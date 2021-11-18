# LOGBOOK5
## M08G04
- João Sousa
- Rafael Ribeiro
- Xavier Pisco

## Buffer Overflow Attack Lab (Set-UID Version)

### Task 1: Getting Familiar with Shellcode

> *Run them and describe your observations.*

A new shell seems to be launched, despite some visual bugs with deleting characters. It is the zsh that we previously set as the /bin/sh.

![](https://i.imgur.com/R6qBg39.png)

If we make this programs Set-UIDs programs (by running `make setuid` instead of `make`), we have access to the zsh as the root user instead.

![](https://i.imgur.com/G3jUQiQ.png)

### Task 3: Launching Attack on 32-bit Program (Level 1)

Following the instructions provided by running gdb on the `stack-L1-gdb` program:
![](https://i.imgur.com/y830YbM.png)

Placing breakpoint before the `bof()` function call:
![](https://i.imgur.com/SmRC0e7.png)

This will make it so the execution "stops before the `ebp` register is set to point to the current stack frame", which is why we call next so the pointer is correctly pointing to the stack frame of the function call.
![](https://i.imgur.com/zR4afSz.png)

The resulting stack base pointer address and the starting address for the `buffer` variable
![](https://i.imgur.com/W6DJOJ2.png)

The difference between these is **108** bytes, and since the return address field is 4 bytes above `ebp`, the real distance between the buffer and the return address ins **112**

In sum, the payload will be composed of:

- Large number of `0x90` bytes (or *NOPs*, which cause the program to continue execution. If the `ret` address undershoots the starting position of the shellcode, these will make it so the program eventually reaches this shellcode)
- Shellcode inserted right at the end of the buffer (starts at `len(buffer) - len(shellcode)`)
- Value to overwrite the return address field. 
    - This value will aproximately point to our malicious shellcode. The value for this `ret` will be the closest guess we have (`ebp + L` => Return address position in the stack ; `ebp + 2L` => Start of NOP codes, first available position to jump to). However, taking into account *Note 2*, we must aim a little deeper in the stack, otherwise we run the risk of undershooting our guess and not reaching deep enough inside the stack to the start of the NOP codes. Thus, a smarter guess has to be in the range [`ebp + 2L`, ???]. Different values can be used here for different users.
    - The position of this address in our payload must replace the current return address in the stack, and thus will be at the difference between the return address' value and the buffer address' value, which we previously calculated to be 112.


![](https://i.imgur.com/SHtUJO7.png)

![](https://i.imgur.com/Ys17ARM.png)

## CTF


### Desafio 1

#### Analyse the code

The given C file opens and reads a file that is specified in the `meme_file` variable, so, if we can change it's value we can open another file.

In that file, `buffer` has **20** bytes of size and, in the `scanf` function **28** bytes are read to it, so, there are **8** bytes that overflow and can be read to another memory space. Since the `meme_file` variable was defined just before the `buffer`, the overflow will change the file that will be open.

#### Exploiting

After understanding the code we know that we just need to send to the program a string with 20 bytes, this can be whatever we want, and the string "flag.txt" to overwrite the `meme_file` variable and, thus, open the *flag.txt* file and get the flag.

The flag we got was flag{28c75ec90c0b6c3022b7e315c2ff7645}

### Desafio 2

#### Analyse the code

After the changes, the C code had one more variable with size **4** bytes defined between the other **2** variables. Since the bytes read are now **32**, it's still possible to overflow the buffer and overwrite both the `val` and the `meme_file` variables.

Going down the code we see that the `val` needs to have a specific value for the program to read the file, and thus this value must be preserved in the stack before the new, injected file name "flag.txt"

This can be bypassed by simply writing the same **20** random bytes, followed by `0xfefc2122`, that **must be written in reverse** and the name of the file we want to read, "flag.txt".

The reversal of the bytes is needed because of the nature of stacks and x86 systems. Stacks are written in reversed order (high to low), and x86 systems are *Little Endian*, which means the least significant byte is in lower memory. Thus, to write something onto the stack that has more than one byte, this procedure is needed. This wasn't evident before because we were only writting strings and converting them into bytes with `.encode()`, which converts the string to `UTF-8`, that is a length oriented format, which means endianness has no meaning in decoding it.


[Source](https://security.stackexchange.com/questions/82750/why-are-buffer-overflows-executed-in-the-direction-they-are)

#### Exploiting

Based on the previous point, we just need to send a messsage containing 12345678901234567890\x22\x21\xfc\xfeflag.txt

As a response we get `flag{becfdcefca03c650d8e05b5ed2006f2f}`
