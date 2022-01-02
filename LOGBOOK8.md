# LOGBOOK8

## Tasks

### Task 1: Get Familiar with SQL Statements

> After running the commands above, you need to use a SQL command to print all the profile information of the employee Alice. Please provide the screenshot of your results.

Since we knew the table name was `credential` and that we wanted all the available information, by running the query `SELECT * FROM credential` we got the following results:

![](https://i.imgur.com/DD5ep2p.png)

### Task 2.1: SQL Injection Attack from webpage.

> Your task is to log into the web application as the administrator from the login page, so you can see the information of all the employees.

Based on the provided code, it would be hard to try and do sql injection through the password, we would need to be able to control the `hashed_pwd` variable, which is a hashed version of our input.

Thus, our attack would be made through the username. If we set the username as `admin` and ignore the password completely we can log in. To ignore the password we need to be able to change the query to end with `WHERE name= ’admin’;` so, if we set the username as `admin';#` we comment out the password verification and, thus, we log in as admin. In this case `#` makes the rest of the query line become a comment.

After that we get this page:
![](https://i.imgur.com/Plu7Tbj.png)


### Task 2.2: SQL Injection Attack from command line.

> Your task is to repeat Task 2.1, but you need to do it without using the webpage.

To perform this attack from the command line, we used the curl tool and we encoded the username used in the previous task to be usable in a URL, which resulted in this command: `curl 'www.seed-server.com/unsafe_home.php?username=admin%27%3B%23&Password='`. When ran, we got the HTML response of the webpage in the terminal which contained a table with all the information about the clients.

![](https://i.imgur.com/7dZrHOE.png)

After some more experimenting, we found that we don't need to have the Password field in the URL as with our attack it isn't used in the PHP code.

### Task 2.3: Append a new SQL statement.

>There is a countermeasure preventing you from running two SQL statements in this attack. Please use the SEED book or resources from the Internet to figure out what this countermeasure is, and describe your discovery in the lab report.

After some search we found, in this [php documentation](https://www.php.net/manual/en/mysqli.quickstart.multiple-statement.php) that the query function used in this example cannot perform more than 1 query to the server, if we wanted to perform multiple ones we would need to use the multi_query function, thus, we cannot perform the login and change the database at the same time.

### Task 3.1: Modify your own salary.

> Please demonstrate how you can achieve that. We assume that you do know that salaries are stored in a column called salary.

To achieve this we just need to add `', salary='40000` after any of the input fields. For example, we can set our NickName to `Alice', salary='40000` and our salary will be 40000.

![](https://i.imgur.com/ESy85QC.png)

### Task 3.2: Modify other people’ salary.

> After increasing your own salary, you decide to punish your boss Boby. You want to reduce his salary to 1 dollar. Please demonstrate how you can achieve that.

To change other peoples' salary, we need to change our input to end with `WHERE Name='Boby'`, thus we input `Boby', salary='1' WHERE Name='Boby';#` in the NickName field, which will set Boby as the NickName for Boby and will set his salary to 1.

![](https://i.imgur.com/G2BQWSh.png)



## CTF Semanas 8&9

### Desafio 1

By looking at the code, and *SQL Injection* vulnerability is evident, since the query's details are being inserted by the user via `POST`. 

![](https://i.imgur.com/Yd2SMkr.png)

By giving the string `' or ''=''` for both username and password fields, the query that is executed is:

```
SELECT username FROM user WHERE username ='' or ''='' AND password ='' or ''=''
```

With this, the query will return all rows from the user table, since the expression `OR '='` is always true. The fact that this string alone worked to get the flag is probably a fluke, since a more correct one would be: 

```
SELECT username FROM user WHERE username ='' or ''='' and username='admin'  AND password ='' or ''=''
```

From this we got the flag, printed out above the login form: 
`flag{e4241c1f70311a853aa80655da83ac14} `

### Desafio 2

By investigating the site, the first aspect that pops out is the ping functionality, that seems to return the output of the linux utility `ping`.

To confirm that there was a change of injecting commands into this fuctionality, we tried giving it the string `8.8.8.8 && ls`, which successfully returned both the output of pinging the `8.8.8.8` address, as well as the contents of the server directory:

![](https://i.imgur.com/aM3vN5B.png)

Thus, to obtain the flag that we knew was located at `/flag.txt`, all that was neeeded was to inject the command `cat /flag.txt` into the field.

![](https://i.imgur.com/pxYnfiU.png)

This yielded the flag `flag{fcc77dd848c93705bdb6e45bf0c50ffd}`