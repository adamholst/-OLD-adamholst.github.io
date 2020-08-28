---
title: THM | Git Happens
image: /images/gh-title.png
date: 2020-08-27 18:00:00 -4:00
categories: [CTF, THM]
tags: [git, gittools, command line fu, github] 
---

# Description
Boss wanted me to create a prototype, so here it is! We even used something called "version control" that made deploying this really easy!
> Difficulty: Easy
>
> Link: [https://tryhackme.com/room/githappens](https://tryhackme.com/room/githappens)

# [Task 1]  Capture the Flag - Can you find the password to the application?

- [ ] Find the Super Secret Password

## Information Gathering
There's a lot of clues of what you will be doing on this box before even starting. It's still good practice to start with a port scan to see what's open, but the first thing I did was go to the site and check for a git directory.

![](/images/gh-gitdir.png)

If you were lost on how to find this, nmap will find it for you after your scan.
```console
PORT   STATE SERVICE VERSION                                                  
80/tcp open  http    nginx 1.14.0 (Ubuntu)
| http-git:                           
|   10.10.106.82:80/.git/                                                     
|     Git repository found!
|_    Repository description: Unnamed repository; edit this file 'description' to name the...
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Super Awesome Site!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## GitTools
So there is an exposed git directory, and a good tool to use to pull everything down is [GitTools](https://github.com/internetwache/GitTools)

To install it, go to a directory you want to put the tool in, I always use /opt. Now just clone the repo on your machine.

![](/images/gh-gtools.png)

You want to use the dumper script to pull down the git directory.

![](/images/gh-dumper.png)

`[*] USAGE: ./gitdumper.sh http://target/.git/ dest-dir`

## git log
Now use the git command line tools to enumerate more information. Start with getting the logs by using `git log`

![](/images/gh-gitlog.png)

So the important thing here is now we have the hashes associated with each commit. Using `git show <hash>` you can pull everything that was commit'd to the repo.

![](/images/gh-gitshow.png)

## Command Line Fu
You could manually type this command for each commit, but that's crazy. Let's use some simple command line utilities to get everything we need in one line. Remember all we want is the commit hash from the first `git show` command, so pipe the log command with `grep commit`

![](/images/gh-grep.png)

Looking better, now we'll expand on this command by cutting out the hash and getting rid of the rest. Pipe this to `cut -d " " -f2`

**Command Breakdown:**

Argument | Description
:------------: | :-------------:
cut | remove sections from each line of files (or raw output)
-d | field delimiter
" " | use spaces as the delimiter
-f2 | select only the content in the second field

![](/images/gh-cut.png)

Okay, so with our command `git log | grep commit | cut -d " " -f2` we get output of just the commit hashes. The only thing left to do is feed this to the very first command `git show` and that will show all the info for every commit made to the repo. Pipe the above command with `xargs git show` xargs will take the output we have so far and turn it into input for the `git show` command

Scrolling through to the very first initial commit, you will find a very basic and insecure version of the website and in the source code is the admin password in plain text.

![](/images/gh-source.png)

- [X] Find the Super Secret Password

This is the answer for the task and you can login to the portal page with admin credentials. 