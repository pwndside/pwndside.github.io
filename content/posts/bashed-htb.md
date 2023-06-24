---
title: "Bashed"
date: 2023-06-24T23:52:47+02:00
tags: [""]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
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

- Room name: Bashed
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Bashed

![Untitled](/HTB/bashed-icon.png)

## Tools Used

- Nmap

## Port Scanning

![Untitled](/HTB/bashed-1.png)

In the port scan we can see that the only service running is a web server.

## Enumeration

![Untitled](/HTB/bashed-4.png)

![/Untitled](/HTB/bashed-5.png)

If we visit the website there are only one article with a web reverse shell information called phpbash, and a url to the github project.

In the url we can find some useful information like how works, this shell removes the necessity of do a traditional reverse shell.

Let’s try this but first some fuzzing

![Untitled](/HTB/bashed-6.png)

We can see that there is a uploads directory maybe we can try to upload a reverse shell.

With nikto, I found that the /dev directory maybe its useful

![Untitled](/HTB/bashed-7.png)

Inside the directory, there are two files, the two of them are the reverse shell which we have seen in the previous github project (seems like a reverse shell to his own machine). If we try to run one we get in.

## **Gaining an Initial Foothold**

![Untitled](/HTB/bashed-8.png)

We are logged like the web service and the user flag is in the arrexel home directory.

![Untitled](/HTB/bashed-9.png)

Now let’s elevate privilege

If we go to the / directory we can see that inside there is a script folder created by scriptmanager user.

When we try sudo -l we found that the the www-data user has acces to scriptmanager as sudo.

Let’s try `sudo -u scriptmanager -e /bin/bash` 

![Untitled](/HTB/bashed-10.png)

Don’t seem to work, maybe the web server doesn’t allow some commands.

Now let’s try to upload the reverse shell and this time trying to connect it to our machine instead of using the shell on /dev.

One way is to create a http server and curl the file in the victim machine. We need to modify the reverse shell file that I get [here](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) with our ip and port.

![Untitled](/HTB/bashed-11.png)

![Untitled](/HTB/bashed-12.png)

Now let’s open a listening port with netcat and run the shell.

![Untitled](/HTB/bashed-13.png)

![Untitled](/HTB/bashed-14.png)

Before nothing, we are going to pass from a dumb shell to a fully interactive one.

First once we spawn a bash with `python -c 'import pty; pty.spawn("/bin/bash")` then lets keep the nc in background with ctrl^z, once done we use `stty raw -echo` and then `fg` to return to the web shell but fully interactive this time.

![Untitled](/HTB/bashed-14-2.png)

Let’s retry `sudo -u scriptmanager -e /bin/bash`

![/HTB/bashed-15.png](/HTB/bashed-15.png)

We are scriptmanager now, let’s see what we can do in the /scripts folder

## Lessons Learned

- What did you learn from the room?
    
    
- Were there any new tools or techniques used?

**Thank you for reading, and happy hacking! 😄**