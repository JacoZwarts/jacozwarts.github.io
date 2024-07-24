---
layout: post
title:  "PortSwigger - Authentication - Lab 13"
date:   2024-06-06 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication]
categories: [PortSwigger]
---

# Lab 13 - Broken brute-force protection, multiple credentials per request
><b>Lab Objective:</b>
<br/><br/>
This lab is vulnerable due to a logic flaw in its brute-force protection. To solve the lab, brute-force Carlos's password, then access his account page.<br/>
`Victim's username: carlos`<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">Candidate passwords</a>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request">Port Swigger - Lab 13</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, investigate the login page. Notice that the `POST /login` request submits the login credentials in `JSON` format. Send this request to Burp Repeater.

#### 2. n Burp Repeater, replace the single string value of the password with an array of strings containing all of the candidate passwords. For example: `"username" : "carlos", "password" : [ "123456", "password", "qwerty" ... ]`

#### 3. Send the request. This will return a `302` response.

#### 4. Right-click on this request and select `Show response in browser`. Copy the URL and load it in the browser. The page loads and you are logged in as `carlos`.

#### 5. Click `My account` to access Carlos's account page and solve the lab
<hr/>