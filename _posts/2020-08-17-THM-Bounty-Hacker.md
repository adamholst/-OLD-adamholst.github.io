
---
title: THM Bounty Hacker
date: 2020-08-17 16:54:00 +/-TTTT
categories: [CTF Walkthrough, Pen Test]
tags: [ctf, thm]     # TAG names should always be lowercase
---

# Information Gathering
### Nmap
```
adam@harambe:~$ nmap -sVC -v 10.10.219.20
```
```
PORT      STATE  SERVICE         REASON       VERSION
21/tcp    open   ftp             syn-ack      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.11.75
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open   ssh             syn-ack      OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCgcwCtWTBLYfcPeyDkCNmq6mXb/qZExzWud7PuaWL38rUCUpDu6kvqKMLQRHX4H3vmnPE/YMkQIvmz4KUX4H/aXdw0sX5n9jrennTzkKb/zvqWNlT6zvJBWDDwjv5g9d34cMkE9fUlnn2gbczsmaK6Zo337F40ez1iwU0B39e5XOqhC37vJuqfej6c/C4o5FcYgRqktS/kdcbcm7FJ+fHH9xmUkiGIpvcJu+E4ZMtMQm4bFMTJ58bexLszN0rUn17d2K4+lHsITPVnIxdn9hSc3UomDrWWg+hWknWDcGpzXrQjCajO395PlZ0SBNDdN+B14E0m6lRY9GlyCD9hvwwB
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMCu8L8U5da2RnlmmnGLtYtOy0Km3tMKLqm4dDG+CraYh7kgzgSVNdAjCOSfh3lIq9zdwajW+1q9kbbICVb07ZQ=
|   256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICqmJn+c7Fx6s0k8SCxAJAoJB7pS/RRtWjkaeDftreFw
80/tcp    open   http            syn-ack      Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
From the above output we learn that ports **21**, **22**, and **80** are open. FTP being open is the first thing I noticed, and more importantly, that anonymous login is allowed. Let's go check it out.

# FTP

![alt text](https://i.imgur.com/FWnPcYL.png)

There are two .txt files being hosted on the ftp server:
**locks.txt** - contains a list of complex passwords.
**task.txt** - contains arbitrary tasks, but is signed by Lin (username)
With these two pieces of information it is now possible to attempt to bruteforce the ssh login.

# SSH
### Hydra
![alt text](https://i.imgur.com/wDSgYZ1.png)
Using the username and password list with hydra confirms valid SSH credentials to gain an initial foothold. 
# User Flag

```
lin@bountyhacker:~/Desktop$ ls -la
total 12
drwxr-xr-x  2 lin lin 4096 Jun  7 17:06 .
drwxr-xr-x 19 lin lin 4096 Jun  7 22:17 ..
-rw-rw-r--  1 lin lin   21 Jun  7 17:06 user.txt
lin@bountyhacker:~/Desktop$ cat user.txt
THM{REDACTED}
```
First flag found just like that. Now, at this point there's many ways to start poking around to see how you are going to **escalate your privileges**. Since I have the current user's password, I like to check their **sudo permissions**. 

### Priv Esc:
```
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```
I know that the tar binary has a sudo priv esc over at [GTFOBins](https://gtfobins.github.io/)
![alt text](https://i.imgur.com/A9UEJAD.png)
ez one-liner ftw??
![alt text](https://i.imgur.com/TEz5F6j.png)
Yup
# Root Flag
Only one thing left to do so let's check the usual location of the flag.
```
root@bountyhacker:~/Desktop# cd /root
root@bountyhacker:/root# ls -la
total 40
drwx------  5 root root 4096 Jun  7 21:31 .
drwxr-xr-x 24 root root 4096 Jun  6 06:36 ..
-rw-------  1 root root 2694 Jun  7 22:25 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  2 root root 4096 Feb 26  2019 .cache
drwxr-xr-x  2 root root 4096 Jun  7 15:00 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   19 Jun  7 17:16 root.txt
-rw-r--r--  1 root root   66 Jun  7 21:13 .selected_editor
drwx------  2 root root 4096 Jun  7 19:29 .ssh
root@bountyhacker:/root# cat root.txt
THM{*REDACTED*}
```

That's it, machine rooted. 