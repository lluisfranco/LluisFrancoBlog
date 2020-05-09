---
title: "Check if current user is domain administrator"
subtitle: "Querying your enterprise domain groups"
date: 2010-12-13T11:51:50+02:00
featuredImage: /images/posts/sysadmin.png
fontawesome: "true"
categories: 
- Development
- How To
tags:
- csharp
- net framework
- API
- Win32
- Active Directory
- Domain
- Administrator
# draft: true
---

## The scenario

In business applications it’s very common checking if the current user is member of the ‘domain administrators’ role of your company. For example, recently I had to check if the current user has administrative privileges in order to show some advanced configuration options.

### IsInRole

To acomplish this we could use the [IsInRole](http://msdn.microsoft.com/en-us/library/system.security.principal.windowsprincipal.isinrole(v=VS.80).aspx) method from the [WindowsPrincipal](http://msdn.microsoft.com/en-us/library/system.security.principal.windowsprincipal(VS.80).aspx) class. This method checks if an user is member of a Windows role and returns a bool value. One of its overrides allows to pass the SID of the role or a constant value based on the enumeration [WindowsBuiltInrole](http://msdn.microsoft.com/en-us/library/system.security.principal.windowsbuiltinrole(v=VS.80).aspx).

> Note: For performance reasons, it’s recommended to use the override: IsinRole(SecurityIdentifier).

To check if current user is an administrator on the local computer we only need to do this:

{{< highlight csharp "linenos=table" >}}
var wp = new WindowsPrincipal(WindowsIdentity.GetCurrent());
return wp.IsInRole(WindowsBuiltInRole.Administrator);
{{< / highlight >}}

Notice that it’s realy easy, but WindowsBuiltInrole enumeration only contains local roles. So, if we would check if our user is member of a domain group, we should find the role SID in our domain.

### WellKnownSidType

Let’s take a look at the following enumeration [WellKnownSidType](http://msdn.microsoft.com/en-us/library/system.security.principal.wellknownsidtype.aspx), this enumeration provides commonly used security identifiers and that's exactly whan we want. Let’s try to use it in our code:

{{< highlight csharp "linenos=table" >}}
var wp = new WindowsPrincipal(WindowsIdentity.GetCurrent());
var sid = new SecurityIdentifier(WellKnownSidType.AccountDomainAdminsSid, null);
return wp.IsInRole(sid);
{{< / highlight >}}

{{< figure src="/images/posts/currentuserisdomainadminerror.png" title="Boom!" >}}

Do’h! It seems that we need to pass the second argument called DomainSId, which represents the security identifier of your company domain.

### DomainSid

This domain SID **is required for some WellKnownSidType values** and we can get it using the [DirectoryEntry](https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices.directoryentry?view=dotnet-plat-ext-3.1) class from the assembly [System.DirectoryServices](https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices?view=dotnet-plat-ext-3.1).

{{< highlight csharp "linenos=table" >}}
var domain = Domain.GetDomain(new DirectoryContext(
    DirectoryContextType.Domain,
    IPGlobalProperties.GetIPGlobalProperties().DomainName));
using (DirectoryEntry de = domain.GetDirectoryEntry())
{
    var domainSIdArray = (byte[])de.Properties["objectSid"].Value;
    var domainSId = new SecurityIdentifier(domainSIdArray, 0);
}
{{< / highlight >}}

First, we obtain a reference to the domain using the domain name. Then get of the value the property objectSid as a byte array, and finally  transform this value into a valid SecurityIdentifier which is what we need.

## Show me the code

Putting it all together. Like my collegue and friend [@alegrebandolero](https://twitter.com/alegrebandolero) I’m also a fan of extension methods. So, let’s create an extension method for the [WindowsIdentity](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.windowsidentity?view=dotnet-plat-ext-3.1) class:

{{< highlight csharp "linenos=table" >}}
public static bool IsDomainAdmin(this WindowsIdentity identity)
{
    var domain = Domain.GetDomain(new DirectoryContext(
        DirectoryContextType.Domain,
        IPGlobalProperties.GetIPGlobalProperties().DomainName));
    using (DirectoryEntry de = domain.GetDirectoryEntry())
    {
        var domainSIdArray = (byte[])de.Properties["objectSid"].Value;
        var domainSId = new SecurityIdentifier(domainSIdArray, 0);
        var domainAdminsSId = new SecurityIdentifier(
        WellKnownSidType.AccountDomainAdminsSid, domainSId);
        var wp = new WindowsPrincipal(identity);
        return wp.IsInRole(domainAdminsSId);
    }
}
{{< / highlight >}}

That’s all. Now, use it as follows:

{{< highlight csharp "linenos=table" >}}
if(WindowsIdentity.GetCurrent().IsDomainAdmin())
{
    //some code here...
}
{{< / highlight >}}

> Edit 12/14/2010: Since Windows Vista each Windows user have a couple of security tokens. The first one is the normal token with limited privileges, and the second one only works when you ‘run as administrator’. This code only works if you are using the second token, running the application as administrator.

HYEI, happy coding!

December 2010
