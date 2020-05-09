---
title: "Remove diacritics (accents) in strings"
subtitle: "In an elegany way :P"
date: 2014-06-26T11:51:50+02:00
featuredImage: /images/posts/note.png
fontawesome: "true"
categories: 
- Development
- How To
tags:
- csharp
- net framework
- Tip
- SQL
# draft: true
---

## Quick note

How to remove diacritics (accents) from a C# string?

{{< highlight csharp "linenos=table" >}}
public static string RemoveDiacritics(this string input)
{
    var stFormD = input.Normalize(NormalizationForm.FormD);
    var len = stFormD.Length;
    var sb = new StringBuilder();
    for (int i = 0; i < len; i++)
    {
        var uc = System.Globalization.CharUnicodeInfo.GetUnicodeCategory(stFormD[i]);
        if (uc != System.Globalization.UnicodeCategory.NonSpacingMark)
        {
            sb.Append(stFormD[i]);
        }
    }
    return (sb.ToString().Normalize(NormalizationForm.FormC));
}
{{< / highlight >}}

## Bonus track

How to remove in TSQL (Microsoft SQL Server)

{{< highlight tsql "linenos=table" >}}
SELECT 'àéêöhello!' Collate SQL_Latin1_General_CP1253_CI_AI
{{< / highlight >}}

See you soon! :wink:

