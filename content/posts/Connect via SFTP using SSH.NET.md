---
title: "Connect via SFTP using SSH.NET"
subtitle: "Secure FTP? Really?"
date: 2017-11-29T11:51:50+02:00
featuredImage: /images/posts/ftp.png
fontawesome: "true"
categories: 
- Development
- How To
tags:
- csharp
- net framework
- SFTP
- SSH.NET
# draft: true
---
## Overview

In these times of APIs-everywhere, it may sound like an anachronism the use SFTP to connect to a remote server and get a list of files for exchanging information, but in the financial world (sadly) it’s more common than you think.

In my current project I’ve to connect to a remote server via Secure File Transfer Protocol (aka SFTP) using a user name, a RSA private key and a phassphrase. Once connected, the goal is to read some files from a remote folder and download them to a local folder.

To accomplish this I recommend you to use SSH.NET, one of the most popular SSH libraries for .NET, available on nuget.

{{< figure src="/images/posts/sshnet.png" title="SSH.NET on NuGet" >}}

More info about SSH.NET [here](https://github.com/sshnet/SSH.NET/).

## Connecting to the server

The most difficult part here is configure the connection, as usual. We need to provide the server url and port, in combination with the username and a file that contains the private key of the RSA certificate, and -of course- the passphrase.

Here’s the final code, and it works :)

{{< highlight csharp "linenos=table,hl_lines=7" >}}
private static void SFTP_Connect_And_Download_Sample()
{
    var host = "sftp_server_address.com";
    var port = 999;
    var username = "your_username";
    var passphrase = "your_passphrase";
    var privateKeyLocalFilePath = @"your_localpath\...\PrivateKeyOpenSSH.ppk";

    var remoteFolderPath = "/";
    var localFolferPath = @"D:\locafiles\";

    var key = File.ReadAllText(privateKeyLocalFilePath);
    var buf = new MemoryStream(Encoding.UTF8.GetBytes(key));
    var privateKeyFile = new PrivateKeyFile(buf, passphrase);
    var connectionInfo = new ConnectionInfo(
        host, port, username,
        new PrivateKeyAuthenticationMethod(username, privateKeyFile));
    using (var client = new SftpClient(connectionInfo))
    {
        client.Connect();
        var files = client.ListDirectory(remoteFolderPath);
        foreach (var file in files)
        {
            Console.WriteLine(file);
            using (var targetFile = new FileStream(
                Path.Combine(localFolferPath, file.Name), FileMode.Create))
            {
                client.DownloadFile(file.FullName, targetFile);
                targetFile.Close();
            }
        }
        client.Disconnect();
    }
}
{{< / highlight >}}
{{< admonition tip "Note" >}}
Use the full path to your file in the **privateKeyLocalFilePath** variable.
{{< /admonition >}}

Take a look at the highlighted line (7). The key here is creating a file that contains the privateKey. I recommend you to use filezilla or similar to test the connection and save it to a file. Your file should looks something like this:

{{< highlight systemd "linenos=table" >}}
PuTTY-User-Key-File-2: ssh-rsa
Encryption: aes256-cbc
Comment: rsa-key-KBL-20171006
Public-Lines: 6
AAB3NzaC1yc2EAAAJQAAAQEAnHp1CA4xF04ZdOQ/rsxJoW9fPJ2RD
FgMNVIqsUsjeRbIoZ2y8SMD9b7MMB0lpKXgJ2dYDgOnh2q
j4VTpEoI2JWh4NdQgSH0O+2oLmQnwgDPT7Kva095ggEQiqScX4+31aY02/nz
mK86sxq/sUsW/UqgS+pPViRQLVzDXFf8XIYSSZngmV+Rk108BQ==
Private-Lines: 14
khHfZWB0vBIFtKc4s98xGDFhwZNJQByUTtE7um0tcU4cwy1QTPf3GIuN
vyhHxIGx3LBtFJqzbZVJtOJVSYjkBTGbNc62D0uJCxYVf8PUUStI6GbOkkyDW/Vt
beWZ3s/DugsImjcbPxdEz/2X86uzd5U5v4/wGKQr8GWJtNksMcJ
k7JgRYGA/t0cSE2980MxhZOBg2Gn7+0A6mWgSf2Rr7hpcqsou1
hmc1HYtN7Oj4WT7hvRt8ZAC3/ekTdJ4K3K7vKglSHoQ
tnimHaanJHz4RGBb78Alllk+OYk3TN0Etcwod3401cLpjYYeq6veZLA/KfCHuiJ+
+Zqoy//NY9egfXd1hB0kmiemwO8wGfLS7ppS/WvPOknW9I8SNMllR1vmO3Hk6S3x
KfAG03ZWNoKDLvAIUllNyMpf9p8oKLF2ny4bJKsfNtr4Y0ejQJUFkC
uNz1S4VJ8j1fRcIjx4yT121B9BDfp486RUmnEgsOFEtmVyHPfNxYzDXq2MjTf4l/
MI5cKWHrDbDC/Cu3YvkF3gekLAb/j9Cie/feHmSnbuZ2VEr6zUt10yaH4hPejCOw
FYDZb0I8xxxWJZ6BbpLWDqeD0Oiss8UnDhha/iKvodA9LIG0T
VRlScvGvuzClGYkc7UIWIoARvdxp46YlGMu4mWGeVkNcrxXnmUkdKyNqGjAGJoK/
Private-MAC: d288fffe72914eb62ed60a7e50fd8ce775
{{< / highlight >}}

## Bonus (trick)

I’ve spent a lot of time before it works receiving a SSH exception “Invalid private key file”, but the same key file works fine when using the app Filezilla to connect, so #WhatTheHell is going on :grimacing:

Finally I realized this is because the private key must be compatible with SshNet, so we have to convert the private key using PuTTY key Generator (in my case, the same app we used to create the certificate). Once opened in PuTTY, just export your file key to OpenSSH and use this new ppk file instead of the previous one.

{{< figure src="/images/posts/exportosk.png" title="Export OpenSSH key" >}}

If you open the new file you will see that the key file now begins with:

{{< highlight systemd "linenos=table" >}}
—–BEGIN RSA PRIVATE KEY—–
{{< / highlight >}}

And ends with:

{{< highlight systemd "linenos=table" >}}
—–END RSA PRIVATE KEY—–
{{< / highlight >}}

Now this is a valid SSH private key and it shoud work like a charm ;)

Hope you enjoy it!
