---

title: Active Directory Lab Build
date: 2020-10-14 16:00:00 +5:30
description: The following post is a guide for setting up an Active Directory environment
image: /images/adchrome.jpg
categories: [Tutorials, Lab Builds]
tags: [Active Directory, AD, Domain Controller, DC, Westeros, lab, tutorial, vmware, iso, windows 10, windows server 2019] # add tag

---

# <ins>What is Active Directory?</ins>
Active Directory (AD) is a directory service developed by Microsoft for Windows domain networks. It stores and maintains information related to all users and computers in a Windows domain type network. A server running Active Directory Domain Service (AD DS) role is called a domain controller (DC). It authenticates and authorizes all users and computers in a Windows domain type network—assigning and enforcing security policies for all computers and installing or updating software.

Think of Active Directory as the contacts app on your phone. In your contacts list you have a bunch of different people that have an assortment of information associated with them. So then, when you get a phone call or a text from one of them, the contacts app can then associate their number with a name, or blocks them if they are annoying. You can give other apps access to your contacts list to sync up and to connect with them on some other app. AD works likes this too, called forming trusts between domains.
>*tl;dr - AD stores information about objects on the network and makes it easy for administrators and users to find and use. AD uses a structured data store for the logical, hierarchical organization of directory information.*

# <ins>Why Active Directory?</ins>
AD is used by 95% of the Fortune 1000 companies. So if you work in IT, you will encounter AD at one point. Understanding the fundamentals of how to use AD will help you understand many other technologies that are a part of AD or closely associated with it.

Its popularity is why adversaries are so enticed to discover and exploit vulnerabilities within the Active Directory ecosystem. In order to defend against those types of attacks, there is a need for practice grounds where Pen Testers, Security Researchers, and Ethical hackers can practice offensive and defensive methodologies.

# <ins>Lab Overview</ins>
- 1 Windows Server 2019
- 2 Windows 10 Enterprise Edition
- 60 GB of Disk Space
- 16 GB RAM
- VMware or VirtualBox to use as a hypervisor.
	- I'm going to be using VMware in this guide. 

This will be enough to set up one domain with a DC and two workstations. We'll also create a few extra user accounts, a service account, set up a group policy, and create a file share.

![](/images/adlab3.png)

I like Game of Thrones, so I themed my domain as such. Having some sort of theme can help to remember what's what in the domain sometimes, rather than just having PC1, PC2, user1, etc.

# <ins>Download ISOs</ins>
- [Windows 10 Enterprise](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise)
	- 100% free, the evaluation license for this image is good for 90 days. However, it will still work just fine after the 90 days have expired.
- [Windows Server 2019](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019)
	- Same as the Windows 10 image, but the license is good for 180 days. Will work after the expired date just fine too.

When downloading the ISOs you will be required to fill out a form asking for your information, you don't have to put your real information. You will not receive a confirmation email or anything like that.

# <ins>Create The Domain Controller</ins>

## Create New VM
1. Open VMware and select "Create a New Virtual Machine". Browse to the ISO of the Windows Server 2019 that was downloaded and select it.
2. Select the drop down for the standard version and leave everything else blank, including the product key. Click next, and click "ok" when prompted about the product key.
3. Leave everything default and click next all the way to the last window. Before you click finish make sure to deselect the "Power on this VM after creation" then click finish.

## Edit VM Settings
1. Click 'Edit VM settings' on the VMware dashboard. 
2. Select the floppy drive and click remove. The floppy drive will prevent us from installing so just remove it.
3. Default memory should be 2GB of RAM, if not change it to that.
4. Make sure the network adapter is NAT.

## Play VM
1. Click the play virtual machine button.
2. Get ready and when the screen says 'press any key' hit a key, otherwise you will miss the windows setup process and have to restart your machine.

## Windows Setup
1. Select your language and click install now.
2. Select Windows Standard Edition (Desktop Experience).
3. Accept the terms and click custom install.
4. Select new and apply the defaults to create the partitions.
5. Finish installing and create a password for your admin account.

## Done? Nope
1. Need to change just a few things.
2. Search for 'PC name' and change the PC name to the DC name of your domain. Mine will be `IRONTHRONE-DC`.
3. Restart the PC when prompted.

## Install AD DS 
1. When server manager loads, click manage in the top right and select add roles/features.
2. Leave defaults and click next until you get to the 'Server Roles' screen.
3. Select the box next to 'Active Directory Domain Services' and click next.
4. Leave defaults and click next all the way to the install button. Click install.

## Promote To DC
1. You will now see a flag at the top right of the server manager window.
2. Click it and select 'promote this server to a DC'.
3. Select add a new forest.
4. Provide the root domain name. Mine will be `WESTEROS.local`.
5. Provide a password on the next screen, you can use the same one as before if you'd like.
6. Keep clicking next and leave everything default until you get to the install screen. Click install.
7. The DC will restart automatically once it's done installing.
8. Everything is done on the DC for now, so we can move on to the user workstations.

# <ins>Setup the User Workstations</ins>

## Create New VM
1. Open VMware and select "Create a New Virtual Machine". Browse to the ISO of the Windows 10 client that was downloaded and select it.
2. Select the drop down for the Windows 10 Enterprise version and leave everything else blank, including the product key. Click next, and click "ok" when prompted about the product key.
3. Leave everything default and click next all the way to the last window. Before you click finish make sure to deselect the "Power on this VM after creation" then click finish.

## Edit VM Settings
> This is the same as the DC from above.

1. Click 'Edit VM settings' on the VMware dashboard. 
2. Select the floppy drive and click remove. The floppy drive will prevent us from installing so just remove it.
3. Default memory should be 2GB of RAM, if not change it to that.
4. Make sure the network adapter is NAT.

## Play VM
> This is the same as the DC from above.

1. Click the play virtual machine button.
2. Get ready and when the screen says 'press any key' hit a key, otherwise you will miss the windows setup process and have to restart your machine.

## Windows Setup
1. Select your language and click install now.
2. Accept the terms and click custom install.
3. Select new and apply the defaults to create the partitions.
4. Select your region, keyboard, skip the additional keyboard.
5. When you are asked to make an account, stop and select 'Domain join instead' in the lower left of the screen.
6. Create you user name, I'm using Jon Snow and make a password.
7. Fill out the security questions, click no when asked about accessing more devices, and disable the long list of spyware.
8. And finally hit accept.

## Done? Nope
1. Need to change just a few things.
2. Search for 'PC name' and change the PC name to its domain name. Mine will be `WINTERFELL`.
3. Restart the PC when prompted.
> Repeat all these steps for the second workstation.

# <ins>Configure the DC</ins>
## Users
1. Login to the domain admin account from earlier.
2. In server manager, click Tools -> Active Directory Users and Computers.
3. Expand WESTEROS.local and select the users folder.
4. Right click in the white space and create a new user.
5. I'm going to make the Jon Snow account.
	- This is different from the one made earlier, that one is a local user on that specific machine. This will be an AD user and be able to log in to any machine in the domain.
6. Use a consistent naming convention for the logon name. Mine will be first initial last name -> `jsnow`.
7. Next, make a password. Change password to 'never expires' and deselect the rest.
8. Make an additional domain admin account.
9. Right click the administrator account and click copy.
10. Fill out the user info just like above. Mine will be Daenerys Targaryen.
11. Create another regular user, right click Jon Snow and select copy.
12. This will be the Jamie Lannister account from my diagram.
13. Now we will do a no-no. Create a SQL service account with domain admin permissions.
14. Right click the Daenerys Targaryen user and select copy.
15. I will name mine SQLService.

## File Share
1. On the server manager dashboard, select 'File and Storage Services' located on the left side.
2. Click on shares, and on the top of the windows there's a 'TASKS' drop down. Click new share.
3. Select the default, SMB Share - Quick and click next all the way to the share name window.
4. Name the share whatever you want, mine is hackme.
5. Leave all defaults and click next all the way to the create button and create the share.
All of this opens up ports 139,445 and gives us an opportunity to get hands-on with SMB.

## Service Principle Name (SPN)
This needs to be configured for kerberos to work so we can explore kerberoasting.
1. Open a CMD prompt as administrator.
2. Type the command:
```console
setspn -a IRONTHRONE-DC/SQLService.WESTEROS.local:60111 WESTEROS\SQLService
```

## Group Policy (GPO)
1. In the search bar, type group policy and run 'Group Policy Management' as administrator.
2. Expand the domains folder, right click the folder and select 'Create a GPO...'
3. Name it 'Disable Windows Defender'
> AV evasion is important, but it's such a technical step in the attack process and TTPs change almost every patch Tuesday so it really doesn't matter for what I want to use this lab for. You will find it is quite common to disable AVs when researching vulnerabilities.

4. Right click the newly created policy and select edit.
5. Expand to Computer Configuration -> Policies -> Administrator Templates -> Windows Components -> Windows Defender Antivirus
6. Double click 'Turn Off Windows Defender Antivirus'
7. Select enable, apply, ok. Defender is now turned off for all PCs connected to our domain.

# <ins>Join All the Things</ins>
Now it's time to join the workstations to the DC and some users to the workstations.
## Join Workstations
1. On the DC, open a CMD prompt and type `ipconfig` to get the IP address. We will need this shortly.
2. Login to the Winterfell PC and use the Jon Snow local account.
3. Right click the network icon on the bottom right of the taskbar and click 'Open Network & Internet Settings'
4. Select change adapter options. Right click Ethernet0 and select properties.
5. Double click the IPv4 option. Click the bubble to select a static IP for the DNS address.
6. Paste in the IP address of the DC.
7. Use the taskbar search, and search for domain. Select 'Access work or school'.
8. Click connect, and at the bottom of the window select 'Join this device to a local Active Directory domain'.
9. Enter WESTEROS.local as the domain and login as the Administrator. Click skip on the add an account window. Go ahead and restart now.
10. Login as other user and login as jsnow. It should sign in just fine, now sign out and login as WESTEROS\Administrator.
11. Right click the start icon and click computer management. Expand local users and groups -> click the groups folder -> double click Administrators.
12. Add jsnow as an administrator.

> Now follow the same steps for the other workstation and also add jsnow AND jlannister and administrators for the KINGSLANDING computer.

## DC Final Look
1. Log back into the DC and go back to the 'Active Directory Users and Computers' window.
2. Refresh the computers folder and you should see both workstations.
