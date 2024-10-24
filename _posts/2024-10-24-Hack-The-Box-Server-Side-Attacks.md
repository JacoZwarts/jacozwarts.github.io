---
layout: post
title:  "Hack The Box - Academy - Server Side Attacks"
description: "Follow this in-depth walkthrough of Hack The Box Academy's Server Side Attacks module. Learn how to exploit SSRF, SSTI, SSI, and XSLT vulnerabilities step-by-step, and enhance your penetration testing skills with expert insights from Zwarts Sec."
date:   2024-10-24 23:36
image:  /images/htb/server-side-attacks/logo.png
tags:   [server-side-attacks,ssrf,ssti,ssi,xslt,cbbh]
categories: [htbacademy]
---

>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/145/section/1293">Server Side Attacks</a><br/>
<a href="https://jacozwarts.github.io/images/htb/server-side-attacks/Server_Side_Attacks_Module_Cheat_Sheet.pdf">Server Side Attacks - Cheat Sheet</a><br/>
<a href="https://github.com/tarunkant/Gopherus">Gopherus</a><br/>
<a href="https://github.com/vladko312/SSTImap">SSTImap</a><br/>
<br/>

<hr/>

## Notes:
### Server Side Template Injection (SSTI):
#### jinja2 Remote Code execution

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

### Install SSTIMap:
```
git clone https://github.com/vladko312/SSTImap
```
`cd into SSTIMap directory`

```
python3 -m venv venv
```

```
source venv/bin/activate
```

```
pip install -r requirements.txt
```

`Usage:`

```
python3 sstimap.py -u http://{TARGET}:{PORT}/index.php?name=test
```
<hr/>

## Skills Assessment
><b>Overview:</b>
You are tasked to perform a security assessment of a client's web application. Apply what you have learned in this module to obtain the flag.

### Inspect the Source Code and Identify JavaScript Functionality

Begin by carefully examining the web application's source code. Pay particular attention to any JavaScript that handles POST requests. In this case, we notice a post request being made via JavaScript.

![Skills Assessment Source Code](/images/htb/server-side-attacks/skills-assessment-source-code.png)

### Intercept the POST Request with Caido or Burp Suite

To investigate further, start Caido or Burp Suite to intercept and analyze the request. You will notice that three POST requests are logged.

![Intercept request with Caido](/images/htb/server-side-attacks/skills-assessment-ssrf-post-request.png)

```
api=http://truckapi.htb/?id%3DFusionExpress01
```

This indicates that the request is being sent to an external API, which could be vulnerable to Server-Side Request Forgery (SSRF).

### Test for SSRF Vulnerability

We can test for SSRF by replacing the external domain (`truckapi.htb`) with a local one. Change the domain to `127.0.0.1:80` and resend the request to see if we get a response from the local server. If we receive a response, the endpoint is likely vulnerable to SSRF.

### Scan for Open Ports

Next, we’ll scan for other open ports on the server. Using Caido’s automation feature, we can perform a port scan to discover additional services. After the scan, we found that `MySQL` is running on port `3306`.

![Skills Assessment Port Fuzzing](/images/htb/server-side-attacks/skills-assessment-caido-port-fuzz.png)

![Skills Assessment Port Fuzz Result](/images/htb/server-side-attacks/skills-assessment-caido-port-fuzz-result.png)

### Try Local File Inclusion (LFI)

With the SSRF confirmed, we can now attempt to access local files. One of the commonly targeted files is `/etc/passwd`, which can give us insight into system users. Use the following payload to check for Local File Inclusion (LFI):

```
file:///etc/passwd
```
![Skills Assessment LFI](/images/htb/server-side-attacks/skills-assessment-lfi-result.png)

### Modify the Payload to Read Specific Files

After confirming that LFI is possible, adjust the payload to read flag.txt file. 

![Caido Flag Result](/images/htb/server-side-attacks/skills-assessment-caido-flag-result.png)
<hr/>