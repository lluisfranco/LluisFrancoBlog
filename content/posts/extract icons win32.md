---
title: "Extract extra large icon from a file, folder or drive"
author: "lluis"
date: 2019-04-16T11:51:50+02:00
draft: false
categories: 
- Development
- How To
tags:
- csharp
- API
- Win32
---

## Extracting system icons from Win32

### Overwiew

Hi, everyone! :)

In a recent project I was required to show a list of files with their associated icons. This task sounds quite easy, and in fact it is… Except if you have to deal with files in network paths or you want to get different icons sizes, apart the typical 32×32.

![some extra large icons](/images/posts/extract_icons_win32.png)

You can achieve this using managed code (the easy way), using the static method [ExtractAssociatedIcon](http://msdn.microsoft.com/en-us/library/vstudio/system.drawing.icon.extractassociatedicon) in the class System.Drawing.Icon, but sadly this method doesn’t work with UNC (Universal Naming Convention) paths nor return other sizes that 32×32 pixels.

I had to show four different icons sizes including the extra-large icon, also called “jumbo”, so I’ve decided to use some functions and structures from the Win32 API. Of course, if anyone knows a better way to do it, please contact with me ASAP :)

### Show me the code

Before starting, let’s take a look to the final code:

#### The final solution

From a file path and the desired size we'll retrieve the associated icon.

{{< highlight csharp "linenos=table, hl_lines=3" >}}
var filename = "\\myNetworkResource\Folder\SampleDocument.pdf";
var size = IconSizeEnum.ExtraLargeIcon;
var image = GetBitmapFromFilePath(filename, size);
{{< / highlight >}}

#### The ingredients

API functions and structures (TODO)

SHGetFileInfoA FUNCTION

{{< highlight csharp "linenos=table" >}}
[DllImport("shell32.dll", SetLastError=true)]
static extern TODO SHGetFileInfoA(TODO);
{{< / highlight >}}

_SHFILEINFOA STRUCTURE

{{< highlight csharp "linenos=table" >}}
typedef struct _SHFILEINFOA {
  HICON hIcon;
  int   iIcon;
  DWORD dwAttributes;
  CHAR  szDisplayName[MAX_PATH];
  CHAR  szTypeName[80];
} SHFILEINFOA;
{{< / highlight >}}

SHGetImageList FUNCTION

{{< highlight csharp "linenos=table" >}}
[DllImport("shell32.dll", EntryPoint = "#727")]
private extern static int SHGetImageList(int iImageList, ref Guid riid, ref IImageList ppv);
{{< / highlight >}}

#### Putting it all together

In this code we’re using several API calls. Here’s the tricky part:

{{< highlight csharp "linenos=table, hl_lines=9 15" >}}
private static IntPtr GetIconHandleFromFilePath(
    string filepath, IconSizeEnum iconsize)
{
    var shinfo = new Shell.SHFILEINFO();
    const uint SHGFI_SYSICONINDEX = 0x4000;
    const int FILE_ATTRIBUTE_NORMAL = 0x80;
    const int ILD_TRANSPARENT = 1;
    uint flags = SHGFI_SYSICONINDEX;
    var retval = SHGetFileInfo(filepath, FILE_ATTRIBUTE_NORMAL,
        ref shinfo, Marshal.SizeOf(shinfo), flags);
    if (retval == 0) throw (new System.IO.FileNotFoundException());
    var iconIndex = shinfo.iIcon;
    var iImageListGuid = newGuid("46EB5926-582E-4017-9FDF-E8998DAA0950");
    Shell.IImageList iml;
    var hres = SHGetImageList((int)iconsize, ref iImageListGuid, out iml);
    var hIcon = IntPtr.Zero;
    hres = iml.GetIcon(iconIndex, ILD_TRANSPARENT, ref hIcon);
    return hIcon;
}
{{< / highlight >}}

First, we need to make call to the [SHGetFileInfo](http://msdn.microsoft.com/en-us/library/windows/desktop/bb762179(v=vs.85).aspx) function that receives a reference to a structure of type [SHFILEINFO](http://msdn.microsoft.com/en-us/library/windows/desktop/bb759792(v=vs.85).aspx), which contains the index of the icon image within the system image list. We will use this index later.

Then we’ve to make is a second call to the [SHGetImageList](http://www.pinvoke.net/default.aspx/shell32.shgetimagelist) function that receives an output parameter with an [IImageList](http://msdn.microsoft.com/en-us/library/windows/desktop/bb761490(v=vs.85).aspx) structure, which is modified within the function.

This struct retrieve a COM interface and we need to keep in mind a couple of things:

a) We must use the GUID of this interface in the declaration:

{{< highlight csharp "linenos=table" >}}
[ComImportAttribute()]
[GuidAttribute(“46EB5926-582E-4017-9FDF-E8998DAA0950”)]
[InterfaceTypeAttribute(ComInterfaceType.InterfaceIsIUnknown)]
{{< / highlight >}}

b) And in the improbable case you are going to deploy your project over XP, remember that SHGetImageList [is not exported correctly in XP](http://support.microsoft.com/default.aspx?scid=kb;EN-US;Q316931). For this reason you must hardcode the function’s entry point. Apparently (and hopefully) ordinal 727 isn’t going to change…

Once we have that COM interface, we only need to call its [GetIcon](http://msdn.microsoft.com/en-us/library/windows/desktop/bb761463(v=vs.85).aspx) method, passing a parameter with the desired size, and obtaining a handle to the icon by reference.

After having the handle its really simple create an Icon from the handle, and then convert to a Bitmap, BitmapSource or other:

{{< highlight csharp "linenos=table, hl_lines=4" >}}
public static System.Drawing.Bitmap GetBitmapFromFilePath(
 string filepath, IconSizeEnum iconsize)
{
    IntPtr hIcon = GetIconHandleFromFilePath(filepath, iconsize);
    var myIcon = System.Drawing.Icon.FromHandle(hIcon);
    var bitmap = myIcon.ToBitmap();
    myIcon.Dispose();
    DestroyIcon(hIcon);
    SendMessage(hIcon, WM_CLOSE, IntPtr.Zero, IntPtr.Zero);
    return bitmap;
}
{{< / highlight >}}

> Tip: It’s very important don’t forget to destroy the resources (Icon) when working with the Win32 API!

This method calls the previous one, obtains the icon’s handle and then creates the icon using the handle. Then creates a bitmap from the icon, destroys the icon and returns the bitmap.

I’ve also created an enumeration IconSizeEnum with the different flags values as well:

{{< highlight csharp "linenos=table" >}}
private const int SHGFI_SMALLICON = 0x1;
private const int SHGFI_LARGEICON = 0x0;
private const int SHIL_JUMBO = 0x4;
private const int SHIL_EXTRALARGE = 0x2;
private const int WM_CLOSE = 0x0010;

public enum IconSizeEnum
{
    SmallIcon16 = SHGFI_SMALLICON,
    MediumIcon32 = SHGFI_LARGEICON,
    LargeIcon48 = SHIL_EXTRALARGE,
    ExtraLargeIcon = SHIL_JUMBO
}
{{< / highlight >}}

Finally, retrieving the icon from a file path it’s as easy as:

{{< highlight csharp "linenos=table" >}}
var size = ShellEx.IconSizeEnum.ExtraLargeIcon;
var ofd = new OpenFileDialog();
if (ofd.ShowDialog() == System.Windows.Forms.DialogResult.OK)
{
    labelFilePath.Text = ofd.FileName;
    pictureBox1.Image = ShellEx.GetBitmapFromFilePath(ofd.FileName, size);
}
{{< / highlight >}}

> You can also use UNC paths, including network paths :)

Hope this helps! ;)
