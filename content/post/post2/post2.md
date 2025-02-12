---
title: "Mr Robot CTF"
date: 2024-11-29T12:10:58+05:30
image: https://w0.peakpx.com/wallpaper/248/580/HD-wallpaper-mr-robot-logo-2018-mr-robot-tv-shows-deviantart-logo.jpg
draft: false
toc: true
tags : [ctf]
---


Mr Robot CTF is a medium level linux based machine offered by TryHackMe. We have to find the keys inorder to complete this challenge.

### Enumeration

#### Nmap Scan

First of we start with scanning the target
```
sudo nmap -sT -T4 -p- -sV 10.10.134.174
```
Nmap results:

```
Nmap scan report for 10.10.134.174
Host is up (0.17s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd

Service detection performed. 
Please report any incorrect results at https://nmap.org/submit/
Nmap done: 1 IP address (1 host up) scanned in 352.56 seconds
```

We have a 2 open ports both of them have a webserver running one has SSL/HTTP at port 443 and the other is HTTP at port 80.

Opening the web app in the browser provides us with an interesting interface in which we aren't interested.

Examining the webapp for common directories I stumbled across the `/robots.txt` endpoint.

Content of which were as follows:
```
User-agent: *
fsocity.dic
key-1-of-3.txt
```
For the first key we have the end point `/key-1-of-3.txt`.

We also found the `/fsocity.dic` endpoint which has a wordlist which might come in handy later.

#### Directory Busting

Now moving on, we need to do directory busting to find new endpoints, we employ DirBuster for this task:

![DirBuster](dirb.png)

This yields us some really interesting results: 
`/license`
`/login`

Visiting the `/license` endpoint we get:

![/license](license.png)

Scrolling further down we are greeted with a base64 string which screams "password"

![/license](string.png)

After decoding the base64 string we found we get not only the password but also the username which quite matches the theme of this box:
User: `Elliot`
Password: `ER28-0652`

Now equipped with user and password we move on to the `/login` endpoint.

![login](login.png)

We login with the credentials we have and we get a wordpress dashboard.

![dashboard](dash.png)

### Exploitation

Poking around I found the theme editorhiding in plainsight, this bad boy could be our way in.

![editor](404.png)

Now we have access to some code that can be manipulated and used to gain a reverse shell on the target, what we need to do is replace the code for 404.php with a reverse shell.

Before we execute the reverse shell file on the target we need to setup a listener on our host.
For that we have netcat.

```
rlwrap nc -lvnp 1234
```
Using rlwrap is not necessary but it's the quality of life stuff. Now we have our listener up and running. 

![listener](nc.png)

Executing our reverse shell by visiting the URL: `http://10.10.134.174/wp-includes/themes/TwentyFifteen/404.php`

We get a shell.

![shell](shell.png)

We land as daemon which is not privileged enough to access the "key" file so we need to change to a system user which in this case is "robot" it's password is given in a md5 hash.

We crack the hash using John the Ripper and the fsocity.dic wordlist we found earlier, now we have the password: `abcdefghijklmnopqrstuvwxyz`

Now inorder to change the user we need to have an interactive shell which right now we don't, so to make the current shell interactive we simply use:

```
python -c 'import pty;pty.spawn("/bin/bash")'
```
And we have an interactive shell

![shell](key2.png)

Now we can easily obtain the second key.