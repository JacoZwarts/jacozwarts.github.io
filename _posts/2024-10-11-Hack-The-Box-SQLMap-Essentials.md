---
layout: post
title:  "Hack The Box - Academy - SQLMap Essentials"
date:   2024-10-11 21:58
image:  /images/htb/sqlmap_essentials/logo.png
tags:   [sqlmap,cbbh]
categories: [htbacademy]
---

><b>Overview:</b>
You are given access to a web application with basic protection mechanisms. Use the skills learned in this module to find the SQLi vulnerability with SQLMap and exploit it accordingly. To complete this module, find the flag and submit it here.
<br/>

<br/>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/58/section/509">SQLMap Essentials</a><br/>
<a href="https://jacozwarts.github.io/images/htb/sqlmap_essentials/Sqlmap_Essentials_Module_Cheat_Sheet.pdf">SQLMap - Cheat Sheet</a>
<br/>

### Intercepting and Saving Requests via Burp Suite:
1. `Enable FoxyProxy`: Ensure that FoxyProxy is enabled and correctly configured to route traffic through Burp Suite.
1. `Capture the Request`: In Burp Suite, intercept the HTTP request you want to analyze.
1. `Save the Request`: Right-click on the intercepted request and select Copy to File. Choose a location on your system to save the request for later analysis or replay.
![Save Burp Suite Request](/images/htb/sqlmap_essentials/burp-suite-save-request.png)
<hr/>

### Running SQLMap on an HTTP Request - Exercises

#### What's the contents of the table flag2 (Case #2)

<hr/>