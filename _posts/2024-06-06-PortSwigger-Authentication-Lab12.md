---
layout: post
title:  "PortSwigger - Authentication - Lab 12"
date:   2024-06-06 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,bscp]
categories: [PortSwigger]
---

# Lab 12 - Password brute-force via password change
><b>Lab Objective:</b>
<br/><br/>
This lab's password change functionality makes it vulnerable to brute-force attacks. To solve the lab, use the list of candidate passwords to brute-force Carlos's account and access his "My account" page.
<br/>
<br/>
`Your credentials: wiener:peter`<br/>
`Victim's username: carlos`<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">Candidate passwords</a>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-brute-force-via-password-change">Port Swigger - Lab 12</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, log in and experiment with the password change functionality. Observe that the username is submitted as hidden input in the request.

#### 2. Notice the behavior when you enter the wrong current password. If the two entries for the new password match, the account is locked. However, if you enter two different new passwords, an error message simply states `Current password is incorrect`. If you enter a valid current password, but two different new passwords, the message says `New passwords do not match`. We can use this message to enumerate correct passwords

#### 3. Enter your correct current password and two new passwords that do not match. Send this `POST /my-account/change-password` request to Burp Intruder.

#### 4. in Burp Intruder, change the `username` parameter to `carlos` and add a payload position to the `current-password` parameter. Make sure that the `new password` parameters are set to two different values. For example: `username=carlos&current-password=§incorrect-password§&new-password-1=123&new-password-2=abc`

#### 5. On the `Payloads` tab, enter the list of passwords as the payload set

#### 6. On the `Settings` tab, add a grep match rule to flag responses containing `New passwords do not match.` Start the attack

#### 7. When the attack finished, notice that one response was found that contains the `New passwords do not match message.` Make a note of this password.

#### 8. In the browser, log out of your own account and lock back in with the username carlos and the password that you just identified

#### 9. Click My account to solve the lab.
<hr/>

### Explained
<hr/>
When entering the correct current password but the new passwords don't match, an error message indicates that the new passwords do not match. Conversely, if the current password is incorrect and the new passwords don't match, the error message states that the current password is incorrect. We can exploit this behavior by brute-forcing Carlos's password while deliberately ensuring the new passwords do not match. We'll look for the response indicating that the new passwords don't match. Once we identify the correct current password, we'll submit matching new passwords, thereby successfully updating Carlos's password

<hr/>