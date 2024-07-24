---
layout: post
title:  "PortSwigger - Authentication - Lab 8"
date:   2024-06-04 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication]
categories: [PortSwigger]
---

# Lab 8 - 2FA broken logic
><b>Lab Objective:</b>
<br/><br/>
This lab's two-factor authentication is vulnerable due to its flawed logic. To solve the lab, access Carlos's account page
<br/>
<br/>
`Your credentials: wiener:peter`<br/>
`Victim's username: carlos`
<br/>
You also have access to the email server to receive your 2FA verification code
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic">Port Swigger - Lab 8</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, log in to your own account and investigate the 2FA verification process. Notice that in the `POST /login2` request, the `verify` parameter is used to determine which user's account is being accessed.

#### 2. Log out of your account.

#### 3. Send the `GET /login2` request to Burp Repeater. Change the value of the `verify` parameter to `carlos` and send the request. This ensures that a temporary 2FA code is generated for Carlos.

#### 4. Go to the login page and enter your username and password. Then, submit an invalid 2FA code.

#### 5. Send the `POST /login2` request to Burp Intruder.

#### 6. In Burp Intruder, set the `verify` parameter to `carlos` and add a payload position to the `mfa-code` parameter. Brute-force the verification code.

#### 7. Load the `302` response in the browser.

#### 8. Click `My account` to solve the lab.

<hr/>