---
title: "Sense"
date: 2023-08-22T12:14:14+02:00
tags: ["privesc","web","linux"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "Linux machine with a website running pfsense which has credentials leak which help us access to the software. Thanks to a command injection we can access as root."
description: "Linux machine with a website running pfsense which has credentials leak which help us access to the software. Thanks to a command injection we can access as root."
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

- Room name: Sense
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Sense

![Untitled](/HTB/sense-icon.png)

## Tools Used

- nmap
- searchsploit
- burpsuite

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
   6   │ PORT    STATE SERVICE  REASON         VERSION
   7   │ 80/tcp  open  http     syn-ack ttl 63 lighttpd 1.4.35
   8   │ |_http-server-header: lighttpd/1.4.35
   9   │ | http-methods:
  10   │ |_  Supported Methods: GET HEAD POST OPTIONS
  11   │ |_http-title: Did not follow redirect to https://10.10.10.60/
  12   │ 443/tcp open  ssl/http syn-ack ttl 63 lighttpd 1.4.35
  13   │ |_ssl-date: TLS randomness does not represent time
  14   │ |_http-favicon: Unknown favicon MD5: 082559A7867CF27ACAB7E9867A8B320F
  15   │ |_http-title: Login
  16   │ | http-methods:
  17   │ |_  Supported Methods: GET HEAD POST OPTIONS
  18   │ |_http-server-header: lighttpd/1.4.35
  19   │ | ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizatio
       │ nName=CompanyName/stateOrProvinceName=Somewhere/countryName=US/organiza
       │ tionalUnitName=Organizational Unit Name (eg, section)/emailAddress=Emai
       │ l Address/localityName=Somecity
  20   │ | Issuer: commonName=Common Name (eg, YOUR name)/organizationName=Compa
       │ nyName/stateOrProvinceName=Somewhere/countryName=US/organizationalUnitN
       │ ame=Organizational Unit Name (eg, section)/emailAddress=Email Address/l
       │ ocalityName=Somecity
  21   │ | Public Key type: rsa
  22   │ | Public Key bits: 1024
  23   │ | Signature Algorithm: sha256WithRSAEncryption
  24   │ | Not valid before: 2017-10-14T19:21:35
  25   │ | Not valid after:  2023-04-06T19:21:35
  26   │ | MD5:   65f8:b00f:57d2:3468:2c52:0f44:8110:c622
  27   │ | SHA-1: 4f7c:9a75:cb7f:70d3:8087:08cb:8c27:20dc:05f1:bb02
  28   │ | -----BEGIN CERTIFICATE-----
  29   │ | MIIEKDCCA5GgAwIBAgIJALChaIpiwz41MA0GCSqGSIb3DQEBCwUAMIG/MQswCQYD
  30   │ | VQQGEwJVUzESMBAGA1UECBMJU29tZXdoZXJlMREwDwYDVQQHEwhTb21lY2l0eTEU
  31   │ | MBIGA1UEChMLQ29tcGFueU5hbWUxLzAtBgNVBAsTJk9yZ2FuaXphdGlvbmFsIFVu
  32   │ | aXQgTmFtZSAoZWcsIHNlY3Rpb24pMSQwIgYDVQQDExtDb21tb24gTmFtZSAoZWcs
  33   │ | IFlPVVIgbmFtZSkxHDAaBgkqhkiG9w0BCQEWDUVtYWlsIEFkZHJlc3MwHhcNMTcx
  34   │ | MDE0MTkyMTM1WhcNMjMwNDA2MTkyMTM1WjCBvzELMAkGA1UEBhMCVVMxEjAQBgNV
  35   │ | BAgTCVNvbWV3aGVyZTERMA8GA1UEBxMIU29tZWNpdHkxFDASBgNVBAoTC0NvbXBh
  36   │ | bnlOYW1lMS8wLQYDVQQLEyZPcmdhbml6YXRpb25hbCBVbml0IE5hbWUgKGVnLCBz
  37   │ | ZWN0aW9uKTEkMCIGA1UEAxMbQ29tbW9uIE5hbWUgKGVnLCBZT1VSIG5hbWUpMRww
  38   │ | GgYJKoZIhvcNAQkBFg1FbWFpbCBBZGRyZXNzMIGfMA0GCSqGSIb3DQEBAQUAA4GN
  39   │ | ADCBiQKBgQC/sWU6By08lGbvttAfx47SWksgA7FavNrEoW9IRp0W/RF9Fp5BQesL
  40   │ | L3FMJ0MHyGcfRhnL5VwDCL0E+1Y05az8PY8kUmjvxSvxQCLn6Mh3nTZkiAJ8vpB0
  41   │ | WAnjltrTCEsv7Dnz2OofkpqaUnoNGfO3uKWPvRXl9OlSe/BcDStffQIDAQABo4IB
  42   │ | KDCCASQwHQYDVR0OBBYEFDK5DS/hTsi9SHxT749Od/p3Lq05MIH0BgNVHSMEgeww
  43   │ | gemAFDK5DS/hTsi9SHxT749Od/p3Lq05oYHFpIHCMIG/MQswCQYDVQQGEwJVUzES
  44   │ | MBAGA1UECBMJU29tZXdoZXJlMREwDwYDVQQHEwhTb21lY2l0eTEUMBIGA1UEChML
  45   │ | Q29tcGFueU5hbWUxLzAtBgNVBAsTJk9yZ2FuaXphdGlvbmFsIFVuaXQgTmFtZSAo
  46   │ | ZWcsIHNlY3Rpb24pMSQwIgYDVQQDExtDb21tb24gTmFtZSAoZWcsIFlPVVIgbmFt
  47   │ | ZSkxHDAaBgkqhkiG9w0BCQEWDUVtYWlsIEFkZHJlc3OCCQCwoWiKYsM+NTAMBgNV
  48   │ | HRMEBTADAQH/MA0GCSqGSIb3DQEBCwUAA4GBAHNn+1AX2qwJ9zhgN3I4ES1Vq84l
  49   │ | n6p7OoBefxcf31Pn3VDnbvJJFFcZdplDxbIWh5lyjpTHRJQyHECtEMW677rFXJAl
  50   │ | /cEYWHDndn9Gwaxn7JyffK5lUAPMPEDtudQb3cxrevP/iFZwefi2d5p3jFkDCcGI
  51   │ | +Y0tZRIRzHWgQHa/
  52   │ |_-----END CERTIFICATE-----
```

We have two open ports, one with **http** and one with **https**.

## Enumeration

When I access to the http page seems to redirect you to the ssl one.

![Untitled](/HTB/sense-1.png)

This will be our only point of entry. If we look at the page there is a pfsense login form.

Investigating on google I found that pfsense is a firewall/router computer software distribution based on FreeBSD. 

Searching for default credentials, I found admin/pfsense.

![Untitled](/HTB/sense-2.png)

I am not able to access with them so maybe they changed the passwords.

Only I can do now is to fuzz in order to obtain something else.

```bash
   1   │ /index.html/          (Status: 200) [Size: 329]
   2   │ /index.php/           (Status: 200) [Size: 6690]
   3   │ /help.php/            (Status: 200) [Size: 6689]
   4   │ /stats.php/           (Status: 200) [Size: 6690]
   5   │ /edit.php/            (Status: 200) [Size: 6689]
   6   │ /license.php/         (Status: 200) [Size: 6692]
   7   │ /system.php/          (Status: 200) [Size: 6691]
   8   │ /status.php/          (Status: 200) [Size: 6691]
   9   │ /changelog.txt/       (Status: 200) [Size: 271]
  10   │ /system-users.txt/    (Status: 200) [Size: 100]
  11   │ /exec.php/            (Status: 200) [Size: 6689]
  12   │ /graph.php/           (Status: 200) [Size: 6690]
  13   │ /tree/                (Status: 200) [Size: 7492]
  14   │ /gui.css/             (Status: 200) [Size: 6590]
  15   │ /wizard.php/          (Status: 200) [Size: 6691]
  16   │ /pkg.php/             (Status: 200) [Size: 6688]
  17   │ /installer/           (Status: 302) [Size: 0] [--> installer.php]
  18   │ /xmlrpc.php/          (Status: 200) [Size: 384]
  19   │ /treeview.css/        (Status: 200) [Size: 726]
```

The scan results gives us two interesting directories /changelog.txt and /system-users.txt

![Untitled](/HTB/sense-3.png)

In /changelog.txt directory we found information about security stuff, basically told us that there is one vulnerability which it isn’t patched yet.

![Untitled](/HTB/sense-4.png)

The /system-users.txt gives us a. credentials, maybe we can try to access with them in the main login page.

## Exploiting

![Untitled](/HTB/sense-5.png)

We have the current software version of pfsense which we are using. Let’s search any exploit.

![Untitled](/HTB/sense-6.png)

We only have one which seems a command injection vuln.

![Untitled](/HTB/sense-8.png)

The exploit seems to automatize the things.

![Untitled](/HTB/sense-7.png)

The script doesn’t worked for me, so I decided to see how works to gain access by myself.

```bash
exploit_url = "https://" + rhost + "/status_rrd_graph_img.php?database=queues;"+"printf+" + "'" + payload + "'|sh"
```

The script basically access to the above url and put a payload with a reverse shell, so let’s make it with burp repeater.

The following requests seems to work, its a basic injection to ensure that we have command injection

![Untitled](/HTB/sense-9.png)

Once insured about that there is a command injection. Let’s see what user is executing the commands, I put a nc listener in order to see the output.

![Untitled](/HTB/sense-10.png)

![Untitled](/HTB/sense-11.png)

Good we are root so let’s put a reverse shell to gain access to the machine.

![Untitled](/HTB/sense-12.png)

Something is not working, maybe some kind of character is not covered

![Untitled](/HTB/sense-13.png)

With the above injection the `/` character is not outputted.

I tried the same injection with a bunch of characters and the ones are not outputted are `/` and `-` 

We can pass this characters octal code to a variable and use the variable in order to execute the reverse shell.

![Untitled](/HTB/sense-14.png)

## Gaining an Initial Foothold

![Untitled](/HTB/sense-15.png)

We owned the system, this machine doesn’t need privesc as we can see.

Let’s cat the **user’s flag**.

![Untitled](/HTB/sense-16.png)

Let’s cat the **root’s flag**.

![Untitled](/HTB/sense-17.png)

**Thank you for reading, and happy hacking! 😄**
