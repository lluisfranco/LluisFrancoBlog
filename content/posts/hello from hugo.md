---
title: "Moved to Hugo :)"
subtitle: "dfsgdg rdgre g"
author: "lluis"
authorLink: "/about/"
date: 2020-05-02T11:51:50+02:00
featuredImage: /images/posts/01.jpg
draft: true
categories: 
- Development
- How To
tags:
- a
- b
---

## Aync patterns

### Overwiew

[99]/[100]

Hi, everyone! :)

Sometimes we need to execute some long-running SQL Server processes (Stored procedures or functions) and it’d be nice to get progress messages, and why not, a percent value in order to show a progress bar in our application.

In the past I’ve used different techniques to archive this, from SQL Assemblies for calling a SignalR server, to execute xp_cmdshell, and so many other esoteric ways.

None of that solved the problem in an elegant way, but well, it worked… more or less…

### Changing the point of view

Today I noticed the SqlConnection class has an InfoMessage event, and it can be used for “Clients that want to process warnings or informational messages sent by the server” which is EXACTLY what I was looking for.

It looks promising but, how can I trigger an error from my SQL?

The answer is really **E-A-S-Y** :)

If you are a TSQL programmer you surely know RAISERROR, which generates an error message in a similar way the throw C# keyword does.

The trick here is its severity argument: It indicates the type of problem encountered by SQL Server, and values under 10 indicate that these errors are not severe. Just warnings or information that SQL server sends to anyone is listening.

And who’s listening? You can imagine…

### Examples

Show me the code:
{{< highlight csharp "linenos=table,hl_lines=8 15-17" >}}
class demo1
{
    public static void execute()
    {
        Console.Clear();
        Console.WriteLine(new String('*', 32));
        Console.WriteLine("** DEMO 1 - SYNC Vs. ASYNC    **");
        Console.WriteLine("** METHOD INVOCATION OVERHEAD **");
        Console.WriteLine(new String('*', 32));
        Console.WriteLine("EXECUTING DEMO...");
        demo();
        Console.WriteLine("EXECUTE AGAIN? (S/N)");
    }

    static void demo()
    {
        var sw = new System.Diagnostics.Stopwatch();
        const int ITERS = 1000000;

        EmptyBody();
        EmptyBodyAsync();

        sw.Restart();
        for (int i = 0; i < ITERS; i++) EmptyBody();
        var syncTime = sw.Elapsed;

        sw.Restart();
        for (int i = 0; i < ITERS; i++) EmptyBodyAsync();
        var asyncTime = sw.Elapsed;

        Console.WriteLine(string.Format("\nSYNC  : {0}\nASYNC : {1}\n\nDiff  : {2:F1}x\n",
            syncTime, asyncTime, asyncTime.TotalSeconds / syncTime.TotalSeconds));

    }

    [MethodImpl(MethodImplOptions.NoInlining)]
    private static void EmptyBody() { }

    [MethodImpl(MethodImplOptions.NoInlining)]
    private static async void EmptyBodyAsync() { }
}
{{< / highlight >}}

Instagram example:
{{< instagram BWNjjyYFxVx hidecaption >}}

Twitter example:
{{< tweet 1245963210735353861 >}}

Eof f
