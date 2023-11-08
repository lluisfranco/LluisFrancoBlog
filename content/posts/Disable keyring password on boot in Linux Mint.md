---
title: "Disable keyring password on boot in Linux Mint"
subtitle: "Auto login in Linux Mint using face recognition"
date: 2023-11-08T11:51:50+02:00
featuredImage: /images/posts/LinuxMintBootKeyring.png
fontawesome: "true"
categories: 
- Linux
- How To
tags:
- OS
- Linux Mint
- Keyring
- Boot
- Disable
- Note2Self
# draft: true
---
## Overview

I use Linux Mint in my Slimbook laptop and one of my favorite tools is Slimbook Face, which allows me to login using face recognition.
Configure it it's super easy and I can login using my pretty face, moreover, I can also SUDO 🤩

{{< figure src="/images/posts/slimbook_face.png" title="Slimbook Face tool" >}}

But from time to time, when loging an anoying message appears 🤦

ENTER PASSWORD TO UNLOCK YOUR LOGIN KEYRING

What the hell is that? And why I cannot continue until I type my password?

## Disabling login keyring password

> Warning: This will make your keyring accessible without a password.

### Using UI

The simplest way is to set the password for the keyring to an empty password -- you will not be prompted for a password then:

- Open Applications -> Accessories -> Password and Encryption Keys
- Right-click on the "login" keyring
- Select "Change password"
- Enter your old password and leave the new password blank

> Important: This will expose all your passwords (e.g. email passwords) that you chose to save in the default keyring to anyone using your computer or having access to your files and is therefore not recommended.

Again, this is something I don't care because I'm using face recognition.

### Using Command Line

Use this command replacing MYPASSWORD for your own password.

{{< highlight bash "linenos=table" >}}
python -c "import gnomekeyring;gnomekeyring.change_password_sync('login', 'MYPASSWORD', '');"
{{< / highlight >}}
