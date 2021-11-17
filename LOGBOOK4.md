# LOGBOOK4
## M08G04
- João Sousa
- Rafael Ribeiro
- Xavier Pisco

## Environment Variable and Set-UID Program Lab

### Task 1: Manipulating Environment Variables

> *Please do the following tasks*
> 
![](https://i.imgur.com/TCBugEv.png)


### Task 2: Passing Environment Variables from Parent Process to Child Process

> *Please compile and run the following program, and describe your observation*

By compiling and running the given code, the output is a text log with all the systems environment variables, like **PWD** and **PATH**.

> *Compile and run the code again, and
describe your observation*

By repeating the same steps from before after changing the code so the child's variables are printed, we get the same output as before, a text log with all the variables listed.

> *Please draw your conclusion*

By comparing both logs with `diff`, we conclude they are equal, and thus the child process inherits the environment variables from the calling parent process, which in turn inherits them from the shell.

![](https://i.imgur.com/VLKaDd8.png)


### Task 3:  Environment Variables and `execve()`

> *Please compile and run the following program, and describe your observation.*

By compiling and running the given code, there is no output.

> Change the invocation of `execve()` in Line ➀ to the following; describe your observation.

By compiling and running the code, the output is a text log with all the environment variables.

> Please draw your conclusion regarding how the new program gets its environment variables.

The new program gets its environment variables by using the `environ` variable that is defined in the `unistd.h` library and includes a list with all the environment variables.

![](https://i.imgur.com/Yy4Sh3s.png)


### Task 4: Environment Variables and `system()`

> *Please compile and run the following program to verify this.*

By comparing the output logs with the ones from the previous task, we can confirm the hypothesis that the `system()` call also inherits the environment variables for the new process.

![](https://i.imgur.com/339c1na.png)


### Task 5: Environment Variable and `Set-UID` Programs

> *Please check whether all the environment variables you set
in the shell process (parent) get into the Set-UID child process. Describe your observation. If there are surprises to you, describe them*

Using the same steps as the tasks before, we can see that the `PATH` and `ANY_NAME` variables are inherited, but the `LD_LIBRARY_PATH` isn't when run on a `SET-UID` process. This weird behaviour can be explained by the fact that this variable is ["used only by the dynamic linker"](https://unix.stackexchange.com/questions/394508/is-it-normal-that-ld-library-path-variable-is-missing-from-an-environment)

![](https://i.imgur.com/j9CmvvR.png)
![](https://i.imgur.com/z1lvF4L.png)



### Task 6: The PATH Environment Variable and `Set-UID` Programs

> *Please compile the above program, change its owner to root, and make it a Set-UID program. Can you get this Set-UID program to run your own malicious code, instead of /bin/ls? If you can, is your malicious code running with the root privilege? Describe and explain your observations.*

If we compile the code to a file named `ls` and change our `PATH` to be `$PWD:$PATH`, that is, if we set the directory with our `ls` program to be the first in the `PATH` environment variable, whenever we try to execute the command `ls`, it will run our code instead and, since the code only has a line that calls itself, it will run in an infinite loop until dash stops it.

To run the code with root privilege, even after making the program a Set-UID program, it was necessary to change the `/bin/sh` to `/bin/zsh`.

![](https://i.imgur.com/v5jmk5P.png)

**Note**: The 0 in the last line corresonds to the execution of the line `system("id -u");` in our script, which prints the id of the user executing the program. 0 corresponds to root.

## CTF

### Recognition

Initially, we looked at the bottom of the page we can find `Built with Storefront & WooCommerce.`, this means this websites uses WooComerce and Wordpress.

After searching for some time we found in the shop, the WordPress Hosting product and, in the aditional information we found the 3 following versions.
- Wordpress: 5.8.1
- WooCommerce plugin: 5.7.1
- Booster for WooCommerce plugin: 5.4.3

On the right side of the website we found 2 different users associated with the website:
- admin
- Orval Sanford

### Vulnerabilities search

With some google searches we found that the Booster for WooComerce plugin had a vulnerability that let an authentication bypass.

### Vulnerability choice

We chose the vulnerability we found. It's name is CVE-2021-34646, so we tried to submit the flag flag{CVE-2021-34646} and it worked.

### Find an exploit

In the exploit-db website we searched for that specific CVE and found an exlpoit [here](https://www.exploit-db.com/exploits/50299)

In the script its explained that, to run this script, we needed to execute the script with a url and a port as the server and a code corresponding to the user.

Since we found 2 users and 1 of them is the admin we chose to try and attack using it.

### Exploiting

We run the script with the right parameters and we got 3 different URLs, one of them worked and we went to http://ctf-fsi.fe.up.pt:5001/wp-admin/edit.php and there was a message with the flag flag{9f86e8033975050cb63d0d85876445b4}.