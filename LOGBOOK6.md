# LOGBOOK6
## M08G04
- João Sousa
- Rafael Ribeiro
- Xavier Pisco

## Format String Attack Lab

### Task 1: Crashing the program

> *Your task is to provide an input to the server, such that when the server program tries to print out the user input in the myprintf() function, it will crash. You can tell whether the format program has crashed or not by looking at the container’s printout.*

In this program our input will be used as the first parameter of the printf function. This means that, if we pass a string with specifiers, the printf function will try to access the corresponding arguments but they will not be defined. The stack position of the arguments can be controled by us by ending the first string with an `\0` and setting the memory value we want as the arguments after that.

To crash the program we used a string with `'%s'` or `'%x'` followed by at least one `'\0'` and, after that, a position in memory that the program cannot access, e.g. 0.

Another successful attempt was using the modifier`'%n'` with any or no other characters (e.x. `'%n\0AnyString'`, `'%n'`). This is a [special string modifier](https://www.geeksforgeeks.org/g-fact-31/) that causes the corresponding `printf()` argument's value to be the number of characters preceding the `'%n'` in the string. 
Since this is causing changes to values in memory pointed by the next value in the stack, the program may crash. 
In the case where we send just `'%n'`, the value that will be changed to **0** (since no more characters were written by `printf()`) will be decided by whatever may be placed in the stack after the program runs, which may or may not be forbidden access.


### Task 2: Printing Out the Server Program’s Memory

> *The goal is to print out the data on the stack. How many %x format specifiers do you need so you can get the server program to print out the first four bytes of your input? You can put some unique numbers (4 bytes) there, so when they are printed out, you can immediately tell.*

At first we thought about writing a lot of %x to see when would our input appear. We knew that when our input would start we would see a lot of repeated bytes corresponding to %x.

With 100 `%x` that kind of output appeared and, we started finding what would be the position of the first byte. After trial and error we found that we need 64 `%x` to print the first byte of our input.

This was the output with 64 times this string `%.8x`

![](https://i.imgur.com/LgWeeYn.png)

> There is a secret message (a string) stored in the heap area, and you can find the
address of this string from the server printout. Your job is to print out this secret message.

By reading the output from the server we can see that the secret message is stored in the address `0x080b4008`.

If we write this as the beginning of our message, the 64th parameter of the printf will be this exact address.

So, we write the address, follow by `%x` 63 times, to consume those parameters, and then, we write a `%s` to print the string that starts at `0x080b4008`.

![](https://i.imgur.com/dL0dHq4.png)

> Só reparei depois que me esqueci de tirar os DDDD que foram usados para debug. Deve-se tirar os DDDD e passar para 63

### Task 3: Modifying the Server Program’s Memory
#### Task 3A: Change the value to a different value 
> Your task is considered as a success if you can change it to a
different value, regardless of what value it may be. 

Using what we found in **Task 1**, we will use the `%n` modifier to change the target to a value of our choosing.

The target's address is 0x080e5068 as we can see in the server output, so the payload will start with this value. There is still the need to empty the stack like we found in the previous task, so we add the (64 - 1) empty `%x.` chars (this is because the address at the start of the payload occupies the first of the 64 arguments. After this, the modifier that will write to the address the number given by the number of chars written up until this point, `%n`. And 'voilà', the variable has changed to the length of the string that was written before, which was a `%x.` char. (TODO é isto right? senão qual é a explicação do `0x12b` ??)

![](https://i.imgur.com/N1FuvOP.png)

![](https://i.imgur.com/Argizgl.png)


#### Task 3B: Change the value to 0x5000
> In this sub-task, we need to change the content of the
target variable to a specific value 0x5000.

To write a specific value to an address , we just need to give the `%n` modificer a string with the length of this value. 

The payload will start out the same as our last but in the final `%x`, we set the precision that we need for the %n to have the correct value.

It can be counted as `0x5000 - len(ADDR) - 62*precision` where `len(ADDR)` is the *length of the address of the variable*  and `precision` is the precision used in the `%x` format specifier, which we used as 8. This should print a string with 0x5000 characters before the `%n` and, thus, the target variable will have the value of 0x5000.

![](https://i.imgur.com/lO4YImC.png)

![](https://i.imgur.com/SDWIaGs.png)


## CTF Semana 6

### Desafio 1

Starting with the `checkseck` we get this output:

![](https://i.imgur.com/sSijfZq.png)

This means we can't do ROC because of the canaries.

By checking the code we see that there is a missuse of the function printf in the line 27.

![](https://i.imgur.com/26RMp0V.png)

The user can write to the variable buffer and that variable will be the first parameter of the printf function, that is, it will be the format.

By exploiting this vulnerability we can make the program print the flag variable.

If we use the gdb program we can check the address of the flag variable.

![](https://i.imgur.com/XZsbjNw.png)

Now we just need to trick the program into printing the string at that address.

For that, we can send a string containing the address bytes in reverse `\x60\xc0\x04\x08` followed by the string format specifier `%s`. This will make the code print the string starting at the address and, thus, printing flag{9abb9d7e861f6d2de1481f3037f7e5c5}.

### Desafio 2

After inspecting the `main.c` file we can see that there is a vulnerability in the 14th line, `print(buffer);`. Since we can control the vaiable buffer, we can control what is printed in the printf call and, thus, we can print and change variables.

By reading the rest of the code we can find that the flag is not loaded to memory but there is a way to get a shell on the server and, from that, we can find the flag. For that, we need to set the value of the variable `key` to `0xbeef`.

With the gdb program we found that the variable `key` is stored in the `0x34c00408` address. By trial and error with `%x` in our code, we found that, the **first** address of the *printf* function is the beginning of the `buffer` variable. This means that the first bytes we write on the buffer will be the first address used, e.g. for a input `"\x11\x22\x33\x44%x"`, the output will print `\x11\x22\x33\x4411223344`. 

With this information, we just need a string that starts with `\x34\xc0\x04\x08` and has '0xbeef' number of characters before the `%n`. That can be done with a `%x` with a given precision. So, if we print an address and set the precision to a high value we can change the value of `key` to the desired one.

In this case, since there will be 2 modifiers in the payload, we need to add a dummy address before the key address, so this one is consumed by the `%.x` modifier. We also need to calculate the precision for this modifier, so the total input length is of value `0xbeef`. Thus, it will be `0xbeef` minus 4 bytes for **each** of address at the start of the payload (`0xbeef - 8`), which equals `0xbee7`, and thus, we will input a string like `"AAAA\x34\xc0\x04\x08%.48871x%n"`.

After sending this input we get a shell and we just need to execute `cat flag.txt` and we get flag{9b44a34e679ece2545a38b6ec62d36a7}


