# LOGBOOK11

## Task 1

The first step to setting up a CA is with a `.conf` file. We copy the default one in `/user/lib/ssl/openssl.cnf` to our working directory, and creating the needed directories and files with `mkdir demoCA, cd demoCA, mkdir certs crl newcerts`, `echo 1000 > serial`, `touch index.txt`, as well as uncommenting the line `#unique_subject = no`.

The next step is generating a self-signed certificate for the CA with the following command, which already specifies the needed values: 
```
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
-keyout ca.key -out ca.crt \
-subj "/CN=www.modelICA.com/0=Model CA LTD./C=US" \
-passout pass:dees
```

To decode the generated certificate, we run `openssl x509 -in ca.crt -text -noout` to decode the public-key certificate and `openssl rsa -in ca.key -text -noout` to decode the CA's private key.

> What part of the certificate indicates this is a CA’s certificate?

The `Subject Key Identifier` and `Authority Key Identifier` match, and in "Basic Contraints" CA is TRUE, indicating it is a self-signed certificate.

![](https://i.imgur.com/9MVtKF2.png)

[Source](https://security.stackexchange.com/questions/93162/how-to-know-if-certificate-is-self-signed/139631)

> What part of the certificate indicates this is a self-signed certificate?

If the `Issuer` and `Subject` fields are equal, the certificate is self-signed.

![](https://i.imgur.com/GKwYThV.png)

> In the RSA algorithm, we have a public exponent e, a private exponent d, (...) 
> 
![](https://i.imgur.com/bYDWgfL.png)

> a modulus n, (...)
> 
![](https://i.imgur.com/anQxqRx.png)


> and two secret
numbers p and q, such that n = pq. Please identify the values for these elements in your certificate and key files.

![](https://i.imgur.com/xQX3h1y.png)




## Task 2

To create the server certificates for our own server, as well as the 2 alternative names needed for later, we run

```
openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.smith2020.com/O=Bank32 Inc./C=US" -passout pass:dees -addext "subjectAltName = DNS:www.smith2020.com, \
DNS:www.smith2020A.com, \
DNS:www.smith2020B.com"
```

*Note: Despite the lab referring that we should use our own name and current year as the server name, we only noticed this detail after completing the entire lab, and as such all the commands and screenshots are a result of using smith2020 as the server name.*

## Task 3

Firstly, to allow the command to copy the extension field to the output certificate, we edit the `openssl.cnf` file (the copy from Task 1) and uncomment the line: `copy_extensions = copy`.

Then we run the command to generate the X509 certificate from the certificate signing request obtained in the previous task.

```
openssl ca -config openssl.cnf -policy policy_anything \
-md sha256 -days 3650 \
-in server.csr -out server.crt -batch \
-cert ca.crt -keyfile ca.key
```

> After signing the certificate, please use the following command to print out the decoded content of the certificate, and check whether the alternative names are included. 
>`openssl x509 -in server.crt -text -noout`

In the *X509v3 extensions* section, the alternative names are listed.

![](https://i.imgur.com/2iqkBex.png)


## Task 4

> Please use the above example as a guidance to set up an HTTPS server for your website. Please describe the steps that you have taken, the contents that you add to Apache’s configuration file, and the screenshots of the final outcome showing that you can successfully browse the HTTPS site.


In this task we setup our own server with the default certificates already inside the container. For this, we must edit the `bank32_apache_ssl.conf` file, either by editing the default VirtualHost or by creating a new one with our configurations. One would need to change the ServerName and ServerAlias fields in this part, though since we used the default server name, no action was done.

![](https://i.imgur.com/aoEO9KT.png)

Running the following commands to enable SSL and configuration file, and then start apache so the server goes live.
w

```
root@6458d7b52a25:/# a2enmod ssl
Considering dependency setenvif for ssl:
Module setenvif already enabled
Considering dependency mime for ssl:
Module mime already enabled
Considering dependency socache_shmcb for ssl:
Module socache_shmcb already enabled
Module ssl already enabled
root@6458d7b52a25:/# a2ensite bank32_apache_ssl
Site bank32_apache_ssl already enabled
root@6458d7b52a25:/# service apache2 start
 * Starting Apache httpd web server apache2
 Enter passphrase for SSL/TLS keys for www.bank32.com:443 (RSA):
 * 
```


> Now, point the browser to your web server (note: you should put https at the beginning of your URL, instead of using http). Please describe and explain your observations. Most likely, you will not be able to succeed, this is because ... (the reasons are omitted here; students should provide the explanation in their lab reports). Please fix the problem and demonstrate that you can successfully visit the HTTPS website.

We can't access the website because the web browser does not trust the CA that signed our certificate. To fix that, we need to add it to the list of trusted certificates.

![](https://i.imgur.com/ODL8Agz.png)

In Firefox we need to go to Preferences -> View Certificates -> Import -> Choose ".../modelCA.crt" and we can then access the website normally.


## Task 5

> Instead of launching an actual DNS cache poisoning attack, we simply modify the victim’s machine’s /etc/hosts file to emulate the result of a DNS cache positing attack by mapping the hostname www.example.com to our malicious web server.

> With everything set up, now visit the target real website, and see what
your browser would say. Please explain what you have observed.

After adding the entry `10.9.0.80 www.example.com` to the /etc/hosts file, when we try to access the website `https://www.example.com` we get the following error:

![](https://i.imgur.com/2zkhMrm.png)

This means that the browser does not trust the certificate that the website has and so the user will know the site is compromised.

## Task 6

> Please design an experiment to show that the attacker can successfully launch MITM attacks on any HTTPS website. You can use the same setting created in Task 5, but this time, you need to demonstrate that the MITM attack is successful, i.e., the browser will not raise any suspicion when the victim tries to visit a website but land in the MITM attacker’s fake website.

In order to redirect accesses to `https://www.facebook.com` onto our website, we add the entry `10.9.0.80 www.facebook.com` in the /etc/hosts file (simulating the results of a DNS cache poisoning attack) and also change the certificate that is being used in the apache server by a newly created one with the extension `-addtext "subjectAltName = DNS:www.bank32.com, DNS:www.bank32A.com, DNS:www.bank32B.com, DNS:www.facebook.com"`

![](https://i.imgur.com/epOc6X7.png)
