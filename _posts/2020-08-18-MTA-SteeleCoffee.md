---

title: Malware Analysis | Qbot
date: 2020-08-18 11:00:00 +0:00
categories: [CTFs & Walkthroughs, Malware Traffic Analysis]
tags: [wireshark, qbot, qakbot, virustotal, pcap] # add tag

---

# Description:
The following post is a walk through of the [2020-04-24 malware traffic analysis exercise.](https://malware-traffic-analysis.net/2020/04/24/index.html)
- Zip archive of the pcap: [2020-04-24-traffic-analysis-exercise.pcap.zip 26.9 MB](https://malware-traffic-analysis.net/2020/04/24/2020-04-24-traffic-analysis-exercise.pcap.zip)
- Zip archive of the alerts: [2020-04-24-traffic-analysis-exercise-alerts.jpg.zip 2.8 MB](https://malware-traffic-analysis.net/2020/04/24/2020-04-24-traffic-analysis-exercise-alerts.jpg.zip)
- Password for all files is **infected**
![alt text](https://i.imgur.com/yIMuorh.jpg)
*Shown above: **Alerts** image from the download link.*

# Scenario
-   LAN segment range: 10.0.0.0/24 (10.0.0.0 through 10.0.0.255)
-   Domain: [steelcoffee.net]()
-   Domain controller: 10.0.0.10 - SteelCoffee-DC
-   LAN segment gateway: 10.0.0.1
-   LAN segment broadcast address: 10.0.0.255

# Questions
1.   Which two clients are Windows hosts, and what are the associated user account names?
2.   Which one of these two Windows clients was infected?
3.   What type of malware was that Windows client infected with?

# Question 1
There are many ways to get this information. For me, I filtered for **nbns** packets on wireshark and quickly noticed two IP addresses on the LAN segment communicating with the DC.

![alt text](https://imgur.com/4sAlLnw.png)

You can quickly enumerate the rest of the information you need to identify the clients.
- 10.0.0.149 - DESKTOP-C10SKPY - alyssa.fitzgerald
- 10.0.0.167 - DESKTOP-GRIONXA - elmer.obrien

# Question 2
The alerts image for this exercise has only one alert that states "malware" while all of the others are various informational or policy alerts. The malware alert is:
- **ET MALWARE Windows executable sent when remote host
claims to send an image M3** 

The alert shows this was triggered by traffic from 119.31.234.40 port 80 going back to 10.0.0.167 over tcp port 51132. Use **tcp.port eq 51132** for your **Wireshark filter**, then follow the TCP stream to confirm this is indeed a Windows executable (EXE) file that was identified by the server as an image.

![alt text](https://i.imgur.com/jWIQbI4.png)

![alt text](https://i.imgur.com/7Cd2Zze.png)
The content type for the file is **image/png**, but is actually being executed as a **Windows executable**. At this point we can assess that the infected Windows client was 10.0.0.167.

# Question 3

Export the file from the pcap file for further analysis. In the export window filter for the file name, **8888.png**.

![alt text](https://i.imgur.com/pfHT6Hq.png)

Now, let's get a hash of the file and run a check on virustotal. I'm simply going to get a sha256 hash of the file from my terminal.

```
adam@harambe:~$ sha256 sum 8888.png
f6210da7865e00351c0e79464a1ba14a8ecc59dd79f650f2ff76f1697f6807b1  8888.png
```

Let's see what [virustotal](https://virustotal.com) tells us.
![alt text](https://imgur.com/FoZHQA9.png)

![alt text](https://imgur.com/HqEVJND.png)

A lot of helpful info from just this one source, and to answer the exercise's last question:
- The client was infected with **Qakbot (Qbot) malware**.
