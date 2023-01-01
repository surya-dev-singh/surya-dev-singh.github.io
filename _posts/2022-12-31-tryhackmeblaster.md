# Tryhackme Blaster

Tags: CVE-2019, Privesc, Windows, hhupd

# 🔭Information Gathering

## ➡️ Rustscan

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled.png)

So There are two ports open 

- Port 80 : HTTP SERVER
- Port 3389 : RDP (remote desktop protocol)

## ➡️ **Nmap**

```bash
root@kali ~ » nmap -p 80,3389 -sC -sV 10.10.187.34 -Pn
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-27 12:07 EST
Nmap scan report for 10.10.187.34
Host is up (0.15s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2022-12-27T17:08:28+00:00
|_ssl-date: 2022-12-27T17:08:33+00:00; +1m06s from scanner time.
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2022-12-26T16:35:30
|_Not valid after:  2023-06-27T16:35:30
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1m05s, deviation: 0s, median: 1m05s
```

So , Nmap shows that the host machine is **************windows**************

## 🔍Port 80 : Web Server Enumeration

## ➡️ Gobuster

Since it just a IIS web server , Lets brute force the hidden directory on the web server

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%201.png)

 

It shows that there is a directory called `/retro`  lets enumerate it

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%202.png)

we get the web interface like this , seems like a WordPress site , lets confirm it with **wappalyzer**

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%203.png)

it confirms that the website is running over WordPress 5.2.1 

I tried to login to website , by visiting endpoint `/wp-admin`  , but it was configured to be only accessible from `[localhost](http://localhost)`  . lets enumerate more !

After struggling for a while I read the line form the content from the one of the post on the website **********ready player one**********

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%204.png)

This gives a hint that the password could be the character name form ready player one , lets google it .

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%205.png)

and we got a possible potential password for a username ************wade .************ lets try to use this credentials on RDP .

# 🐾 Initial Foothold

lets RDP on the machine , with `xfreerdp`  like so :

```bash
xfreerdp /v:10.10.187.34 /u:wade /p:parzival /cert:ignore
```

and we got our first access !! 

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%206.png)

we have our user.txt on the Desktop itself !!

# ✈️ Privilege Escalation

so , there is a wired binary on the Desktop of the wade user , let look up for that on internet, interestingly we found that this binary can be used for privilege escalation.

After reading for a while I found that , its basically UAC bypass trick , huppd.exe is windows signed binary , which has clickable link , which open a browser with SYSTEM level privilege , although we get 404 error , if we try to save the page , we are able to open explorer with SYTEM level privilege , now we can open cmd.exe from the explorer.

## ✅ Escalation step

- run `huppd.exe` with administrative privilege

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%207.png)

- click on `show more details` hyperlink

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%208.png)

- now click on `show information about the publisher's certificate`

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%209.png)

- now click on the hyperlink to verify the publisher’s certificate

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%2010.png)

- now try to save this page with CTRL+S , and open the explorer, and from the explorer open cmd.exe

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%2011.png)

And as you can see , we got the command prompt with NT authority\system . lets try to read the root.txt file on the administrator desktop.

![Untitled](/assets/lib/Tryhackme%20Blaster%206afeafbb5a04415d9a8537462a7b20f2/Untitled%2012.png)

Thank you for reading my writeup !!