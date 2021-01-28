---
title: "Dapper: How to change CommandDefinition parameters values"
subtitle: "Useful when working with Dapper and Commands :)"
date: 2021-01-28T11:51:50+02:00
featuredImage: /images/posts/DapperLogo.png
fontawesome: "true"
categories: 
- Development
- How To
tags:
- csharp
- net framework
- Dapper
- CommandDefinition
- Parameters
# draft: true
---
## Overview

In my current development I'm using Dapper, a light ORM to access the database. Using Dapper is really easy and provides super-fast performance. Here's an example of use:

{{< highlight csharp "linenos=table" >}}
public async Task<List<T>> GetSomeDataAsync<T>(
    DateTime startDate, DateTime endDate,
    CancellationTokenSource sourceToken = null)
{
    if (sourceToken == null) sourceToken = new CancellationTokenSource();
    using (var con = CreateConnection())
    {
        var sql = @"SELECT (your field list) FROM (your table/view/function))(
                    @StartDate, @EndDate)";
        var command = CreateCommand(sql, new
        {
            @StartDate = startDate,
            @EndDate = endDate
        }, cancellationToken: sourceToken.Token);
        return await con.QueryCmdAsync<T>(command);
    }
}
{{< / highlight >}}

In this code I'm querying a table valued function that returns some data between two dates.
Both parameters (startDate and endDate) are declared on the fly as anomynous types and used in the creation of a CommandDefinition object.

The CreateCommand method is just an example, and would be used every time we need to create a command to access the database.

{{< highlight csharp "linenos=table" >}}
public static CommandDefinition CreateCommand(
    string commandText, object parameters = null,
    IDbTransaction transaction = null,
    int? commandTimeout = null,
    CommandType? commandType = null,
    CommandFlags flags = CommandFlags.Buffered,
    CancellationToken cancellationToken = default)
{
    //Here you can apply some custom code before the creation of the command definition
    return new CommandDefinition(
        commandText, parameters, transaction,
        commandTimeout ?? SqlHelper.GetCommandTimeout(),
        commandType, flags, cancellationToken);
}
{{< / highlight >}}

### A hypothetical case

Now imagine you have to iterate through the parameters collection and do something, for example prevent invalid values in some parameters before calling the database. A typical case is in most databases the Date types doesn't match with the .NET Framework DateTime, and we want to prevent out of range exceptions:

{{< highlight systemd "linenos=table" >}}
The SQL Server DateTime type range is from January 1, 1753, through December 31, 9999
NET Framework DateTime struct range is from January 1, 0001, through December 31, 9999
{{< / highlight >}}

So if we send a DateTime parameter with value less than 1753/01/01 to the database we will receive an out of range exception. Wouldn't be nice if we check all datetime values before sending to the database?

### Iterating through an anonymous object

As far as I know the only way of iterate over an anonymous object is using reflection.

{{< figure src="/images/posts/ParametersAnonymousType.png" title="Parameters is an Anonymous type" >}}

Using reflection we can access to the properties of the parameters object.
In that case we can check if the parameter property is a DateTime type, check the value and do whatever we need.

{{< highlight csharp "linenos=table" >}}
var paramsProperties = parameters.GetType().GetProperties();
foreach (var p in paramsProperties)
{
    if (p.PropertyType == typeof(DateTime))
    {
        DateTime dateValue = Convert.ToDateTime(
            parameters.GetType().GetProperty(p.Name).GetValue(parameters));
        //do something
    }
}
{{< / highlight >}}

### Modifying the anonymous object

However, in c# anonymous types are immutable and we cannot change their value via reflection.

Sadly, this code won't work:

{{< highlight csharp "linenos=table" >}}
parameters.GetType().GetProperty(p.Name).SetValue(parameters, newValue));
{{< / highlight >}}

There are different ways to solve that. For example we could use an ExpandoObject, but in this case we'll use a Dapper DynamicParameters object because when creating a CommandDefinition we can use an both, an object or a DynamicParameters.

### Putting it all together

So, to prevent SqlDateTime overflows we only need to iterate through the parameters properties using reflection, check these datetimes values and if the value is less that the minimal sql date, change it. Finally add the parameters to a DynamicParameters collection and that's all.

{{< highlight csharp "linenos=table" >}}
private static DynamicParameters GetDynamicParameters(object parameters)
{
    var sqlMinDate = System.Data.SqlTypes.SqlDateTime.MinValue;
    var newParameters = new DynamicParameters();
    var paramsProperties = parameters.GetType().GetProperties();
    foreach (var p in paramsProperties)
    {
        if (p.PropertyType == typeof(DateTime))
        {
            DateTime dateValue = Convert.ToDateTime(
                parameters.GetType().GetProperty(p.Name).GetValue(parameters));
            if (dateValue <= sqlMinDate.Value) dateValue = sqlMinDate.Value;
            newParameters.Add(p.Name, dateValue);
        }
        else
            newParameters.Add(p.Name,
                parameters.GetType().GetProperty(p.Name).GetValue(parameters));
    }
    return newParameters;
}
{{< / highlight >}}

Now we can use the previous method in the CreateCommand method:

{{< highlight csharp "linenos=table" >}}
public static CommandDefinition CreateCommand(
    string commandText, object parameters = null,
    IDbTransaction transaction = null,
    int? commandTimeout = null,
    CommandType? commandType = null,
    CommandFlags flags = CommandFlags.Buffered,
    CancellationToken cancellationToken = default)
{
    if (parameters != null)
        parameters = GetDynamicParameters(parameters);
    return new CommandDefinition(
        commandText, parameters, transaction,
        commandTimeout ?? SqlHelper.GetCommandTimeout(),
        commandType, flags, cancellationToken);
}
{{< / highlight >}}

This post is just a note to my future self :)

Hope you enjoy it, and feel free to send me your thoughts about this solution.
