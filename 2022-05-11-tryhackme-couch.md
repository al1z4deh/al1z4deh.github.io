---
title: Couch - Writeup
author: Elman Alizadeh
date: 2022-05-11 20:48:00 +0800
categories: [ctf, TryHackMe]
tags: [tryhackme, couch]
---

Today we will take a look at TryHackMe: Couch. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/thm-couch/1.jpg" | relative_url }})


## Questions:

1. Scan the machine. How many ports are open?
2. What is the database management system installed on the server? 
3. What port is the database management system running on?
4. What is the version of the management system installed on the server?

You will find the answers to the first four questions during the network scan.

## Network scan

Command: nmap -p- -sV -sC -oN nmap/open — open 10.10.218.153


![Desktop View]({{ "/assets/img/thm-couch/2.png" | relative_url }})


## What is the path for the web administration tool for this database management system?

This requires research.

[CouchDB](https://book.hacktricks.xyz/network-services-pentesting/5984-pentesting-couchdb)

Answer: _utils

![Desktop View]({{ "/assets/img/thm-couch/3.png" | relative_url }})


## What is the path to list all databases in the web browser of the database management system?

When we look at the site I mentioned, we come across such a sentence.

“*/_all_dbs**Returns a list of all the databases in the CouchDB instance.”

Answer: _all_dbs

## What are the credentials found in the web administration tool?

When we return to CouchDB, we see the secret folder. If we look at the content, we can find the username and password in passwordbackup.

![Desktop View]({{ "/assets/img/thm-couch/4.png" | relative_url }})


## Ssh

Let’s connect to the server via ssh with the given credentials.

Command: ssh atena@10.10.218.153

![Desktop View]({{ "/assets/img/thm-couch/5.png" | relative_url }})


## Escalate privileges

When we look at user logs, we come across such a command. Let’s use.

Command: cat .bash_history

![Desktop View]({{ "/assets/img/thm-couch/6.png" | relative_url }})


Command: docker -H 127.0.0.1:2375 run — rm -it — privileged — net=host -v /:/mnt alpine

![Desktop View]({{ "/assets/img/thm-couch/7.png" | relative_url }})


### note: it was normal that you could not find root.txt. because it is at the root in the mnt folder. Good luck;)

And now we are the root

![Desktop View]({{ "/assets/img/thm-couch/1.gif" | relative_url }})


