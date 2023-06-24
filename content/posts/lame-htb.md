---
title: "Lame"
date: 2023-06-24T11:30:03+00:00
tags: [""]
categories: ["hackthebox"]
author: "Ayman Boulaich"
image: "/HTB/lame-icon.png"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This is a friendly Linux machine with a vulnerable Samba service that can be exploited."
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

- Room name: Lame
- Difficulty level: Easy
- Room link: [https://app.hackthebox.com/machines/Lame](https://app.hackthebox.com/machines/Lame)

![Untitled](/HTB/lame-icon.png)

## Tools Used

- Nmap
- Searchsploit
- Metasploit

## Code Review & Exploitation

Upon running Nmap, I discovered three interesting open ports: SSH, FTP, and Samba.

![Untitled](/HTB/lame-1.png)

The FTP service is vsftpd 2.3.4, and my initial attempt was to connect anonymously. I managed to gain access, but it seemed limited. However, exploring through FTP did not yield much information.

![Untitled](/HTB/lame-2.png)

In Searchsploit, I found a potential exploit for vsftpd 2.3.4.

![Untitled](/HTB/lame-3.png)

Unfortunately, this exploit did not work effectively, so I decided to focus on the Samba service.

![Untitled](/HTB/lame-4.png)

I attempted the "username map script" exploit ([CVE-2007–2447](http://cvedetails.com/cve/cve-2007-2447)), which appeared to be the most promising way to authenticate.

![Untitled](/HTB/lame-5.png)

Finally, we gained access.

## Flag(s)

User’s flag

![Untitled](/HTB/lame-6.png)

Root’s flag

![Untitled](/HTB/lame-7.png)

## Lessons Learned

- What did you learn from the room?
    
    I learned how to exploit Samba and gained a basic understanding of machine attack methodology.
    
- Were there any new tools or techniques used?
    
    No new tools were used, but I improved upon my previous knowledge.
    
- What would you do differently next time?
    
    Next time, I would aim to capture better screenshots of my progress. 😅

**Thank you for reading, and happy hacking! 😄**