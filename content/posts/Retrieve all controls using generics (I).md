---
title: "Retrieve all controls using generics (I)"
subtitle: "The easy way"
date: 2010-11-09T17:20:51+02:00
featuredImage: /images/posts/tshirt.png
categories:
- Development
- How To
tags:
- csharp
- net framework
- generics

# draft: true
---

## Gettings all controls from a container

### That's the question

How many times have you ever heard something similar? :)

> How to clear the content of all the textboxes in a form?

The question may vary a littles bit but it's always related to obtain all controls of a determined type (Button TextBox, etc) in a form or maybe in another type of container like a Panel or user control.

### Generics and Extension methods

Using [generics](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/) it’s really easy! Firstly let’s create an [extensor method](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods), that returns a collection of all the controls of a type in a Form or container. And then, we will use this collection to do an action over each returned item.

#### The extensor method

{{< highlight csharp "linenos=table" >}}
public static List<T> GetControls<T>(this Control container) where T : Control
{
    List<T> controls = new List<T>();
    foreach (Control c in container.Controls)
    {
        if (c is T)
            controls.Add((T)c);
        controls.AddRange(GetControls<T>(c));
    }
    return controls;
}
{{< / highlight >}}

This method retrieves the collection of Controls from a container calling it itself recursively, retrieving the content of all his containers.

#### How to use it?

{{< highlight csharp "linenos=table" >}}
this.GetControls<TextBox>().ForEach(p => p.Text = string.Empty);
{{< / highlight >}}

We can  use the extensor method directly into a form because Form class inherits from ContainerControl, that inherits from ScrollableControl, and ScrollableControl from Control. Or, in other words, our Form will implement our extensor method.

So, at compile time we should specify the type of the controls we want to retrieve. In the example we are using TextBox, but you can use Button type instead. And then, use ForEach to apply an action on each one of the items returned.

Moreover, if we want to retrieve only the controls into a container (GroupBox, Panel, TabControl or another that inherits from Control), you can call the method for this particular control.

{{< highlight csharp "linenos=table" >}}
myPanel.GetControls<Button>().ForEach(p => p.Enabled = false);
{{< / highlight >}}

#### Applying several actions

In most cases, we would apply several actions to each control (not only one). In this case it’s quite easy: The only thing we have to do is create a method that receives a parameter of this type, and call it passing the control as an argument:

{{< highlight csharp "linenos=table" >}}
this.GetControls<TextBox>().ForEach(p => ApplyFormat(p));

private void ApplyFormat(TextBox text)
{
    text.BackColor = Color.LightGray;
    text.ForeColor = Color.Red;
    text.Text = "hello";
    text.TextAlign = HorizontalAlignment.Center;
}
{{< / highlight >}}

Or maybe, using an ‘action delegate’, both options are correct:

{{< highlight csharp "linenos=table" >}}
this.GetControls<TextBox>().ForEach(p =>
{
    p.BackColor = Color.LightGray;
    p.ForeColor = Color.Red;
    p.Text = "hello";
    p.TextAlign = HorizontalAlignment.Center;
});
{{< / highlight >}}

Happy coding! :-)

November 2010
