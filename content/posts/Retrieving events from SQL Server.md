---
title: "Retrieving events from SQL Server"
subtitle: "Through the active connection"
date: 2019-09-30T11:51:50+02:00
featuredImage: /images/posts/sqlserver.png
fontawesome: "true"
categories: 
- Development
- How To
tags:
- csharp
- net framework
- SQL
- SQL Server
- Events
# draft: true
---
## Overview

When executing some long-running server side processes on a SQL Server (Stored procedures or functions) would be nice to get progress messages, and why not, a percent value in order to show a progress bar in our application.

In the past I’ve used different techniques to archive this, from SQL Assemblies for calling a SignalR server, to execute [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-2017), and so many other esoteric ways.

None of that solved the problem in an elegant way, but well, it worked… more or less…

## Changing the point of view

Today I noticed the [SqlConnection](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnection?view=netframework-4.8) class has an [InfoMessage](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnection.infomessage?view=netframework-4.8) event, and it can be used for “Clients that want to process warnings or informational messages sent by the server” which is EXACTLY what I was looking for.

{{< figure src="/images/posts/sqlevents.gif" title="Processing SQL events" >}}

It looks promising but, how can I trigger an error from my SQL?

If you are a TSQL programmer you surely know [RAISERROR](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/raiserror-transact-sql?view=sql-server-2017), which generates an error message in a similar way the **throw** C# keyword does.

The trick here is its severity argument: It indicates the type of problem encountered by SQL Server, and [values under 10](https://docs.microsoft.com/en-us/sql/relational-databases/errors-events/database-engine-error-severities?view=sql-server-2017) indicate that these errors are not severe. Just warnings or information that SQL server sends to anyone is listening.

And who’s listening? You can imagine :smile:

## Show me the code

Let’s imagine we’ve this long-running stored procedure in our database. We're using WAITFOR DELAY for simulating intensive processing.

### SQL Script

{{< highlight sql "linenos=table" >}}
CREATE PROCEDURE [dbo].[TestLongRunningSP]
AS
BEGIN
RAISERROR('1) Starting', 0, 0) WITH NOWAIT;
WAITFOR DELAY '00:00:03'; --Wait for 3 seconds
 
RAISERROR('2) Fetching data', 0, 0) WITH NOWAIT;
WAITFOR DELAY '00:00:05'; --Wait for 5 seconds
 
RAISERROR('3) Some other actions', 0, 0) WITH NOWAIT;
WAITFOR DELAY '00:00:03'; --Wait for 3 seconds
 
RAISERROR('4) Calculating', 0, 0) WITH NOWAIT;
WAITFOR DELAY '00:00:02'; --Wait for 2 seconds
 
RAISERROR('5) Finishing', 0, 0) WITH NOWAIT;
END
{{< / highlight >}}

Thanks to RAISEERROR when executing this stored procedure we will show this output.

{{< highlight systemd "linenos=table" >}}
1) Starting
2) Fetching data
3) Some other actions
4) Calculating
5) Finishing

Completion time: 2019-09-30T09:33:28.4952150+02:00
{{< / highlight >}}

Now let’s create some C# code:

### C# code

First: A class to send the received messages from our repository to its consumer, which in this case the UI will be just a Windows Form (good old boys never die).

{{< highlight csharp "linenos=table" >}}
public class SqlMessageEventArgs 
{
    public string Message { get; set; }
    public int Progress { get; set; }
}
{{< / highlight >}}

Notice the Progress property. We will use it later in the last part of the post, be patient :smirk:

Second: A repository class that open a connection, execute the stored procedure and send messages using a ProcessMessage event. We must explicitly enable the connection to receive these errors and raise handling the InfoMessage event:

{{< highlight csharp "linenos=table" >}}
public class SqlServerRepo
{
    public event EventHandler<SqlMessageEventArgs> ProcessMessage;
    public string ConnectionString { get; internal set; }
    public SynchronizationContext SyncContext { get; internal set; }
    public async Task GetDataAsync()
    {
        using (var con = new SqlConnection(ConnectionString))
        {
            con.FireInfoMessageEventOnUserErrors = true;
            con.InfoMessage += (s, e) =>
                SyncContext.Post(_ => ProcessMessage?.Invoke(
                s, new SqlMessageEventArgs
                {
                    Message = e.Message,
                }), null);
            con.Open();
            var sql = "EXEC dbo.TestLongRunningSP";
            var commandDefinition = new CommandDefinition(sql);
            await con.ExecuteAsync(commandDefinition);
            con.Close();
        }
    }
}
{{< / highlight >}}

When using async pattern the received messages come from a different thread and cannot be used to update the UI directly. For this reason I always recommend send to the Repo the current synchronization context and use this Context to post to the main thread.

{{< figure src="/images/posts/threaderror.png" title="Cross thread exception when updating UI from a different thread." >}}

Third: Finally let’s create a simple Windows Form with just a button, a progress bar and a list box).

{{< highlight csharp "linenos=table" >}}
const string connectionstring =
    "Data Source=YOUR_SERVER;" +
    "Initial Catalog=YOUR_DATABASE;" +
    "Integrated Security=True;";

private async void button1_Click(object sender, EventArgs e)
{
    var repo = new SqlServerRepo()
    {
        ConnectionString = connectionstring,
        SyncContext = SynchronizationContext.Current
    };
    repo.ProcessMessage += (ms, me) => {
        listBoxLog.Items.Add($"{DateTime.Now} - {me.Message}");
    };
    await repo.GetDataAsync();
    listBoxLog.Items.Add($"Finished Process!");
}
{{< / highlight >}}

This code creates the repo, pass the connection string and the current synchronization context, handles the ProcessMessage event and invoke the GetDataAsync method.

Simply and elegant. No dependent assemblies or external resources.

### One more thing

What about the progress bar? Just use the state argument of the RAISERROR method on the SP like this:

{{< highlight sql "linenos=table" >}}
CREATE PROCEDURE [dbo].[TestLongRunningSP]
AS
BEGIN
RAISERROR('1) Starting', 0, 10) WITH NOWAIT;
WAITFOR DELAY '00:00:03'; --Wait for 3 seconds
 
RAISERROR('2) Fetching data', 0, 30) WITH NOWAIT;
WAITFOR DELAY '00:00:05'; --Wait for 5 seconds
 
RAISERROR('3) Some other actions', 0, 60) WITH NOWAIT;
WAITFOR DELAY '00:00:03'; --Wait for 3 seconds
 
RAISERROR('4) Calculating', 0, 70) WITH NOWAIT;
WAITFOR DELAY '00:00:02'; --Wait for 2 seconds
 
RAISERROR('5) Finishing', 0, 100) WITH NOWAIT;
END
{{< / highlight >}}

Then update the GetDataAsync method:

{{< highlight csharp "linenos=table, hl_lines=11" >}}
public async Task GetDataAsync()
{
    using (var con = new SqlConnection(ConnectionString))
    {
        con.FireInfoMessageEventOnUserErrors = true;
        con.InfoMessage += (s, e) =>
            SyncContext.Post(_ => ProcessMessage?.Invoke(
            s, new SqlMessageEventArgs
            {
                Message = e.Message,
                Progress = e.Errors[0]?.State ?? 0
            }), null);
        con.Open();
        var sql = "EXEC dbo.TestLongRunningSP";
        var commandDefinition = new CommandDefinition(sql);
        await con.ExecuteAsync(commandDefinition);
        con.Close();
    }
}
{{< / highlight >}}

And finally the code in the form:

{{< highlight csharp "linenos=table" >}}
repo.ProcessMessage += (ms, me) => {
    listBoxLog.Items.Add($"{DateTime.Now} - {me.Message}");
    progressBar.Value = me.Progress;
};
{{< / highlight >}}

And that’s all!

Hope you enjoy it, and please, send me your thoughts about this solution.
