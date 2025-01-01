---
layout: post
title:  "Hack The Box - Academy - Session Security"
description: "Explore this detailed walkthrough of Hack The Box Academy's Session Security module. Learn effective techniques to perform Session Attacks utilizing Session Hijacking, Session Fixation, XSS, CSRF and Open redirects to elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2025-01-01 13:25
image:  /images/htb/session-security/logo.png
tags:   [session-security,cbbh]
categories: [htbacademy]
---

## Explore this detailed walkthrough of Hack The Box Academy's Session Security module. Learn effective techniques to perform Session Attacks utilizing Session Hijacking, Session Fixation, XSS, CSRF and Open redirects.

>
<b>References:</b>
<br/>
<a href="https://academy.hackthebox.com/module/153/section/1389">HTB - Session Security</a><br/>


`Add values to /etc/hosts`
```
IP=ENTER IP ADDRESS
printf "%s\t%s\n\n" "$IP" "example1.net example2.net example3.net" | sudo tee -a /etc/hosts
```