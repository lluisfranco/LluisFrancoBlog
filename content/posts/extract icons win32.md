---
title: "Extract extra large icon from a file, folder or drive"
subtitle: "Including network paths! :)"
date: 2019-04-16T11:51:50+02:00
featuredImage: /images/posts/extra-large-icons.png
fontawesome: "true"
categories: 
- Development
- How To
tags:
- csharp
- net framework
- API
- Win32
# draft: true
---

## Extracting system icons from Win32

### Overwiew

Hi, everyone! :)

In a recent project I was required to show a list of files with their associated icons. It doesn't sounds quite difficult, and in fact it is not. Except if you have to deal with files in network paths or you want to get different icons sizes, apart the typical 32×32. And well, that was the case :(fas fa-grin-beam-sweat):

If you only need local files and sizes of 32x32 píxels you can achieve this using the static method [ExtractAssociatedIcon](http://msdn.microsoft.com/en-us/library/vstudio/system.drawing.icon.extractassociatedicon) in the class System.Drawing.Icon, but sadly this method doesn’t work with UNC (Universal Naming Convention) paths nor return other sizes that 32×32 pixels.

I had to show four different icons sizes including the extra-large icon also called “jumbo”, so I had to some functions and structures from the Win32 API. Of course, if anyone knows a better way to do it, please contact with me ASAP :(fas fa-grin-wink):

### Show me the (final) code

Today I prefer to start from the end so before starting, let’s take a look to the final code:

From a file path and the desired size we wanna retrieve the associated icon.

{{< highlight csharp "linenos=table, hl_lines=3" >}}
var filename = "\\myNetworkResource\Folder\SampleDocument.pdf";
var size = IconSizeEnum.ExtraLargeIcon;
var image = GetFileImageFromPath(filename, size);
{{< / highlight >}}

### The ingredients

Here are declared all the functions and types we need to use. You can simply paste all this code into a class.

{{< highlight csharp "linenos=table" >}}
[ComImportAttribute()]
[GuidAttribute("46EB5926-582E-4017-9FDF-E8998DAA0950")]
[InterfaceTypeAttribute(ComInterfaceType.InterfaceIsIUnknown)]
private interface IImageList
{
    [PreserveSig]
    int Add(
        IntPtr hbmImage,
        IntPtr hbmMask,
        ref int pi);

    [PreserveSig]
    int ReplaceIcon(
        int i,
        IntPtr hicon,
        ref int pi);

    [PreserveSig]
    int SetOverlayImage(
        int iImage,
        int iOverlay);

    [PreserveSig]
    int Replace(
        int i,
        IntPtr hbmImage,
        IntPtr hbmMask);

    [PreserveSig]
    int AddMasked(
        IntPtr hbmImage,
        int crMask,
        ref int pi);

    [PreserveSig]
    int Draw(
        ref IMAGELISTDRAWPARAMS pimldp);

    [PreserveSig]
    int Remove(
        int i);

    [PreserveSig]
    int GetIcon(
        int i,
        int flags,
        ref IntPtr picon);
};
private struct IMAGELISTDRAWPARAMS
{
    public int cbSize;
    public IntPtr himl;
    public int i;
    public IntPtr hdcDst;
    public int x;
    public int y;
    public int cx;
    public int cy;
    public int xBitmap;
    public int yBitmap;
    public int rgbBk;
    public int rgbFg;
    public int fStyle;
    public int dwRop;
    public int fState;
    public int Frame;
    public int crEffect;
}
private struct SHFILEINFO
{
    public IntPtr hIcon;
    public int iIcon;
    public uint dwAttributes;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 254)]
    public string szDisplayName;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]
    public string szTypeName;
}

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

[DllImport("user32")]
private static extern
    IntPtr SendMessage(
    IntPtr handle,
    int Msg,
    IntPtr wParam,
    IntPtr lParam);

[DllImport("shell32.dll")]
private static extern int SHGetImageList(
    int iImageList,
    ref Guid riid,
    out IImageList ppv);

[DllImport("Shell32.dll")]
private static extern int SHGetFileInfo(
    string pszPath,
    int dwFileAttributes,
    ref SHFILEINFO psfi,
    int cbFileInfo,
    uint uFlags);

[DllImport("user32")]
private static extern int DestroyIcon(
    IntPtr hIcon);
{{< / highlight >}}

### Putting it all together

In this code we’re using several API calls to the functions we declared before. You can paste all this code in the same class.
Here’s the tricky part:

{{< highlight csharp "linenos=table" >}}
public static System.Drawing.Bitmap GetFileImageFromPath(
    string filepath, IconSizeEnum iconsize)
{
    IntPtr hIcon = IntPtr.Zero;
    if (System.IO.Directory.Exists(filepath))
        hIcon = GetIconHandleFromFolderPath(filepath, iconsize);
    else
        if (System.IO.File.Exists(filepath))
            hIcon = GetIconHandleFromFilePath(filepath, iconsize);
    return GetBitmapFromIconHandle(hIcon);
}

private static IntPtr GetIconHandleFromFilePath(string filepath, IconSizeEnum iconsize)
{
    var shinfo = new SHFILEINFO();
    const uint SHGFI_SYSICONINDEX = 0x4000;
    const int FILE_ATTRIBUTE_NORMAL = 0x80;
    uint flags = SHGFI_SYSICONINDEX;
    return GetIconHandleFromFilePathWithFlags(filepath, iconsize, ref shinfo, FILE_ATTRIBUTE_NORMAL, flags);
}

private static IntPtr GetIconHandleFromFolderPath(string folderpath, IconSizeEnum iconsize)
{
    var shinfo = new SHFILEINFO();

    const uint SHGFI_ICON = 0x000000100;
    const uint SHGFI_USEFILEATTRIBUTES = 0x000000010;
    const int FILE_ATTRIBUTE_DIRECTORY = 0x00000010;
    uint flags = SHGFI_ICON | SHGFI_USEFILEATTRIBUTES;
    return GetIconHandleFromFilePathWithFlags(folderpath, iconsize, ref shinfo, FILE_ATTRIBUTE_DIRECTORY, flags);
}

private static System.Drawing.Bitmap GetBitmapFromIconHandle(IntPtr hIcon)
{
    if (hIcon == IntPtr.Zero) return null;
    var myIcon = System.Drawing.Icon.FromHandle(hIcon);
    var bitmap = myIcon.ToBitmap();
    myIcon.Dispose();
    DestroyIcon(hIcon);
    SendMessage(hIcon, WM_CLOSE, IntPtr.Zero, IntPtr.Zero);
    return bitmap;
}

private static IntPtr GetIconHandleFromFilePathWithFlags(
    string filepath, IconSizeEnum iconsize,
    ref SHFILEINFO shinfo, int fileAttributeFlag, uint flags)
{
    const int ILD_TRANSPARENT = 1;
    var retval = SHGetFileInfo(filepath, fileAttributeFlag, ref shinfo, Marshal.SizeOf(shinfo), flags);
    if (retval == 0) throw (new System.IO.FileNotFoundException());
    var iconIndex = shinfo.iIcon;
    var iImageListGuid = new Guid("46EB5926-582E-4017-9FDF-E8998DAA0950");
    IImageList iml;
    var hres = SHGetImageList((int)iconsize, ref iImageListGuid, out iml);
    var hIcon = IntPtr.Zero;
    hres = iml.GetIcon(iconIndex, ILD_TRANSPARENT, ref hIcon);
    return hIcon;
}
{{< / highlight >}}

First, we need to call to [SHGetFileInfo](http://msdn.microsoft.com/en-us/library/windows/desktop/bb762179(v=vs.85).aspx) function that receives a reference to a structure of type [SHFILEINFO](http://msdn.microsoft.com/en-us/library/windows/desktop/bb759792(v=vs.85).aspx), which contains the index of the icon image within the system image list. We will use this index later.

Then we’ve to perform a second call to the [SHGetImageList](http://www.pinvoke.net/default.aspx/shell32.shgetimagelist) function that receives an output parameter with an [IImageList](http://msdn.microsoft.com/en-us/library/windows/desktop/bb761490(v=vs.85).aspx) structure, which is modified within the function.

Once we have that COM interface, we only need to call its [GetIcon](http://msdn.microsoft.com/en-us/library/windows/desktop/bb761463(v=vs.85).aspx) method, passing a parameter with the desired size, and obtaining a handle to the icon by reference.

Finally, using the handle we can create an Icon and then convert it to a Bitmap or BitmapSource:

> Tip: Don’t forget to destroy the resources (Icon) when working with the Win32 API!

For better use I’ve also created an enumeration IconSizeEnum (see types declaration) with the different flags values as well:

{{< highlight csharp "linenos=table" >}}
public enum IconSizeEnum
{
    SmallIcon16 = SHGFI_SMALLICON,
    MediumIcon32 = SHGFI_LARGEICON,
    LargeIcon48 = SHIL_EXTRALARGE,
    ExtraLargeIcon = SHIL_JUMBO
}
{{< / highlight >}}

Finally, create a new Form and add a PictureBox named 'pictureBox1' a label called 'labelFilePath' and a Button, and into the Button's click event handler use this code:

{{< highlight csharp "linenos=table" >}}
var size = IconSizeEnum.ExtraLargeIcon;
var ofd = new OpenFileDialog();
if (ofd.ShowDialog() == System.Windows.Forms.DialogResult.OK)
{
    labelFilePath.Text = ofd.FileName;
    pictureBox1.Image = YourClassNameHere.GetFileImageFromPath(ofd.FileName, size);
}
{{< / highlight >}}

> You can also use UNC paths, including network paths :)

Hope this helps! ;)
