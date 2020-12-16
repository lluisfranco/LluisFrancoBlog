---
title: "Macbook Pro - Series 3400 series chipset problems?"
subtitle: "USB and Bluetooth devices"
date: 2011-04-07T11:51:50+02:00
featuredImage: /images/posts/apple.png
fontawesome: "true"
categories: 
- Software
tags:
- Apple
- Boot Camp
- MacBook Pro
- Drivers
- Chipset
- Windows
# draft: true
---

## The issue

After a long time... here we go again!

Today a quick post, just a note:

If you bought a MacBook Pro and using Boot Camp for running Windows, you may be possible experiment some chipset problems. In my particular case with USB and bluetooth devices (Wireless Notebook Presenter 8000), and a couple of window-freezed.

Also, in some ocasions Windows threw an error: "Intel 5 Series/3400 Series Chipset Family USB Universal Host Controller – 3B3B (and 3B36) has a driver problem".

{{< figure src="/images/posts/devicemanager.png" title="Chipset issues on Windows" >}}

## How to fix it

After googling 10 minutes I just realized that lots of people are experimenting the same trouble, and I would like to show how to solve it in 3 simple steps:

1. Open Windows Device Manager
2. Show all devices (including hide devices, View menu)
3. Disable both devices (3B3B & 3B36)

That’s all! :-D

Hope you enjoy this.
