---
title: "Retrieve all controls using generics (II)"
subtitle: "LINQ at rescue!"
date: 2010-11-15T17:20:51+02:00
featuredImage: /images/posts/tshirt2.png
categories:
- Development
- How To
tags:
- csharp
- net framework
- generics
- LINQ

# draft: true
---

## Gettings all controls from a container v2

### A better approach

In my [previous post](/retrieve-all-controls-using-generics-i) I created a recursive function to retrieve all controls inside a form and its containers. Today, my colleague and friend [Eduard Tomàs](https://twitter.com/eiximenis?lang=en) sent me another solution to the same topic based on LINQ. It’s simply and really, really pretty :heart:

### The power of LINQ

Before starting, we need to solve this: LINQ works over [IEnumerable<T>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1?view=netcore-3.1), but Control.Controls property returns a [ControlCollection](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.control.controlcollection?view=netcore-3.1) type. In fact, nowadays, since we have Generics this class has no sense (like other 1.000 similar classes) but remember that Generics doesn’t appeared until .NET Framework 2.0. So, our first step will be retrieve an IEnumerable<Control> from a ControlCollection property of a Control. This could be accomplished using an extensor method over the class ControlCollection:

{{< highlight csharp "linenos=table" >}}
public static IEnumerable<Control> AsEnumerable(this Control.ControlCollection @this)
{
    foreach (var control in @this)
        yield return (Control)control;
}
{{< / highlight >}}

Note that using this method we are able to transform CollectionControl to IEnumerable<Control> and get full access to the power of Linq. Now, let's create a method to retrieve all controls of a type:

{{< highlight csharp "linenos=table" >}}
public static IEnumerable<T> GetAllControls<T>(this Control @this) where T : Control
{
    return @this.Controls.AsEnumerable().Where(
        x => x.GetType() == typeof(T)).Select(
        y => (T)y).Union(
        @this.Controls.AsEnumerable().SelectMany(
            x => GetAllControls<T>(x)).Select(y => (T)y));
}
{{< / highlight >}}

It looks cool, isn’t it? No loops, no ifs… only pure LINQ power! :-) 

Note that converting Controls property into an Enumerable allow us to use all those IEnumerable LINQ methods like [Select](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.select?view=netcore-3.1), [Where](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.where?view=netcore-3.1), [Union](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.union?view=netcore-3.1) or [SelectMany](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.selectmany?view=netcore-3.1)... in just a single line of code! This is amazing, this is functional programming style!

### Applying several actions

There’s also a small difference with my original solution. In the previous one I used an internal List<T> to copy all controls references. The new one only iterates over the original collection. There’s no internal lists.

Another difference present in LINQ solution is we are returning an IEnumerable, so we loose the ForEach method (because IEnumerable doesn’t implements this method). But building our own ForEach method is as trivial as:

{{< highlight csharp "linenos=table" >}}
public static void ForEach<T>(this IEnumerable<T> @this, Action<T> action)
{
    foreach (T t in @this) { action(t); }
}
{{< / highlight >}}

Now, we've all the necessary ingredients to get all controls and do whatever we need :D

{{< highlight csharp "linenos=table" >}}
this.GetAllControls<TextBox>().ForEach(p => p.Enabled = false);

this.GetAllControls<TextBox>().ForEach(p =>
{
    p.BackColor = Color.LightGray;
    p.ForeColor = Color.Red;
    p.Text = "hello";
    p.TextAlign = HorizontalAlignment.Center;
});
{{< / highlight >}}


HYEI, happy coding! :-)

November 2010