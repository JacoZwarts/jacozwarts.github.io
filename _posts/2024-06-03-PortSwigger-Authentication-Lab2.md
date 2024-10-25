---
layout: post
title:  "PortSwigger - Authentication - Lab 2"
date:   2024-06-03 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,bscp]
categories: [PortSwigger]
---

# Lab 2 - Authentication - 2FA bypass
><b>Lab Objective:</b>
<br/><br/>
This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.
<br/>
The lab has an account with a predictable username and password, which can be found in the following wordlists:
<br/>
<br/>
`Your credentials: wiener:peter`<br/>
`Victim's credentials carlos:montoya`
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass">Port Swigger - Lab 2</a>
<br/>

### Solution
<hr/>

#### 1. Log in to your own account. Your 2FA verification code will be sent to you by email. Click the `Email client` button to access your emails.


#### 2. Go to your account page and make a note of the URL.

#### 3. Log out of your account.

#### 4. Log in using the victim's credentials

#### 5. When prompted for the verification code, manually change the URL to navigate to `/my-account`. The lab is solved when the page loads.
<hr/>