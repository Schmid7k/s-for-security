---
title: "Attacktive Directory"
date: 2022-04-04T18:23:05+02:00
draft: false
---

This write up refers to the [Attacktive Directory](https://tryhackme.com/room/attacktivedirectory) room on [TryHackMe](https://tryhackme.com/).

#### Task 1: Deploy the machine

##### Questions 1 - 3)

Deploy the machine attached to this room and connect yourself to the TryHackMe network.

#### Task 2: Setup

Follow the instructions for installing Impacket, Bloodhound and Neo4j.

##### Question 4)

When you have installed the required tools click the `Completed` button to continue.

#### Task 3: Welcome to Attacktive Directory

We begin with some basic enumeration. I'll be using `rustscan` for that though `nmap` is also just fine.

`rustscan -a <ip_address> -- -A -sC -sV`

And we find a whole bunch of open ports

```
PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Simple DNS Plus
80/tcp    open  http          syn-ack Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2022-04-07 12:26:52Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped    syn-ack
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing
47001/tcp open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         syn-ack Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
49670/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49671/tcp open  msrpc         syn-ack Microsoft Windows RPC
49673/tcp open  msrpc         syn-ack Microsoft Windows RPC
49677/tcp open  msrpc         syn-ack Microsoft Windows RPC
49682/tcp open  msrpc         syn-ack Microsoft Windows RPC
49694/tcp open  msrpc         syn-ack Microsoft Windows RPC
49806/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

##### Question 5) What tool will allow us to enumerate port 139/445?

From the enumeration results we found out that port 139 is related to a `netbios-ssn` service and port 445 to a `microsoft-ds` service. These indicate the usage of `samba`.

One particular tool that comes to mind when you want to enumerate samba shares from Linux is `enum4linux`.

**Solution:** enum4linux

##### Question 6) What is the NetBIOS-Domain Name of the machine?

Executing `enum4linux 10.10.1.8` provides us with the information needed to answer this question.

**Solution:** THM-AD

##### Question 7) What invalid TLD do people commonly use for their Active Directory Domain?

This requires some external research.

**Solution:** .local

#### Task 4: Enumerating Users via Kerberos

Download [Kerbrute](https://github.com/ropnop/kerbrute/releases), the [user-list](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) and the [password-list](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt). Make sure to download Kerbrute version 1.0.2 though, because 1.0.3 does not have to UserEnum flag.

##### Question 8) What command within Kerbrute will allow us to enumerate valid usernames?

Executing `kerbrute --help` should give you the answer.

**Solution:** userenum

##### Question 9) What notable account is discovered? (These should jump out at you)

Looking through kerbrute's help menu is everything we need to learn how to use kerbrute. We can enumerate username on the target machine with the following command

`kerbrute userenum userlist.txt --dc <ip_addr> -d domain name`

This gives us a list of usernames, two of which should stick out immediately.

**Solution:** svc-admin

##### Question 10) What is the other notable account is discovered? (These should jump out at you)

**Solution:** backup

#### Task 5: Abusing Kerberos

The intro text tells us about how we are now able to abuse a feature within Kerberos with an attack method called `ASREPRoasting`.
This occurs when a user account has the privilege "Does not require Pre-Authentication" set, meaning that the account does not need to provide valid identification before requesting a Kerberos Ticket.

We can exploit this using `impacket's` tool `GetNPUsers.py` located in `impacket/examples/GetNPUsers.py`, by querying ASRERoastable accounts from the Key Distribution Center. We only need a valid set of usernames, which we have from the `kerbrute` enumeration.

##### Question 11) We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?

Executing `GetNPUsers.py -dc <ip_addr> -no-pass -format john -o hash.txt <domain_name>/<username>` with the correct username should retrieve a hash from the target machine.

**Solution:** svc-admin

##### Question 12) Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)

Searching the [hashcat examples](https://hashcat.net/wiki/doku.php?id=example_hashes) page for the first part of the returned hash leads to the answer.

**Solution:** Kerberos 5, etype 23, AS-REP 

##### Question 13) What mode is the hash?

Executing `hashcat --help` and `grep` for the name of the hash gives us the answer.

**Solution:** 18200

##### Question 14) Now crack the hash with the modified password list provided, what is the user accounts password?

`hashcat -a 0 -m 18200 hash.txt passwordlist.txt` should be able to crack the hash in no time.

**Solution:** man***********

#### Task 6: Back to the Basics

We now have a user's account credentials giving us a lot more access within the domain. Let's proceed by enumerating any shares that the domain controller may be giving out.

##### Question 15) What utility can we use to map remote SMB shares?

Some external research quickly leads to one popular tool for connecting to SMB shares.

**Solution:** smbclient

##### Question 16) Which option will list shares?

`smbclient --help` gives us an overview of common commands and holds the answer to the question.

**Solution:** -L

##### Question 17) How many remote shares is the server listing?

Executing `smbclient -U <username> -L <ip_addr>` prompts us for a password. Luckily this is the password we cracked in question 14!

**Solution:** 6

##### Question 18) There is one particular share that we have access to that contains a text file. Which share is it?

Trying out all the possible shares yields that we have access to the `backup` share. Connecting to it with `smbclient -U <username> \\\\<ip_addr>\\backup` and authenticating with the cracked password allows us to retrieve a file from the share.

**Solution:** backup

##### Question 19) What is the content of the file?

Looking at the contents of the file we retrieved from the backup share shows, that they contain some kind of encoded backup credentials.

**Solution:** YmFja********************************************

##### Question 20) Decoding the contents of the file, what is the full contents?

Giving the encoded string to [cyberchef](https://cyberchef.org/) and using the `Magic` recipe shows that it was encoded using `Base64`.

**Solution:** backup@spookysec.local:*************

#### Task 7: Elevating Privileges within the Domain

The task description tells us that the credentials we just retrieved are related to the backup account of the Domain Controller. This account has a unique permission that allows all Active Directory changes to be synced with this user account, including password hashes.

With this knowledge and the usage of another Impacket tool called `secretsdump.y` we can retrieve all password hashes that his user account has to offer, giving us essentially full control over the AD domain.

##### Question 21) What method allowed us to dump NTDS.DIT?

Executing `secretsdump.py <username>:<password>@<ip_addr>` yields the answer and a lot of hashes.

**Solution:** DRSUAPI

##### Question 22) What is the Administrators NTLM hash?

**Solution:** 0e03*****************************

##### Question 23) What method of attack could allow us to authenticate as the user without the password?

This requires some external research.

**Solution:** Pass the Hash

##### Question 24) Using a tool called Evil-WinRM what option will allow us to use a hash?

As always we can take a look at the help menu by calling `evil-winrm --help`.

**Solution:** -H

#### Task 8: Flag Submission Panel

Now that we basically have full access to the AD domain we can login as admin to get the user flags from their respective desktop folders.

##### Question 25) svc-admin

**Solution:** TryHackMe{#################}

##### Question 26) backup

**Solution:** TryHackMe{###############}

##### Question 27) Administrator

**Solution:** TryHackMe{######################}

Congratulations, you finished the [Attacktive Directory](https://tryhackme.com/room/attacktivedirectory) room on [TryHackMe](https://tryhackme.com/)!

