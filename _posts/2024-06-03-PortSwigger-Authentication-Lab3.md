---
layout: post
title:  "PortSwigger - Authentication - Lab 3"
date:   2024-06-03 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication]
categories: [PortSwigger]
---

# Lab 3 - Authentication - Password reset broken logic
><b>Lab Objective:</b>
<br/><br/>
This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.
<br/>
<br/>
`Your credentials: wiener:peter`<br/>
`Victim's credentials carlos:montoya`
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic">Port Swigger - Lab 3</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, click the `Forgot your password?` link and enter your own username.

#### 2. Click the Email client button to view the password reset email that was sent. Click the link in the email and reset your password to whatever you want.

#### 3. In Burp, go to `Proxy > HTTP` history and study the requests and responses for the password reset functionality. Observe that the reset token is provided as a URL query parameter in the reset email. Notice that when you submit your new password, the `POST /forgot-password?temp-forgot-password-token` request contains the username as hidden input. Send this request to Burp Repeater.

#### 4. In Burp Repeater, observe that the password reset functionality still works even if you delete the value of the `temp-forgot-password-token` parameter in both the URL and request body. This confirms that the token is not being checked when you submit the new password.

#### 5. In the browser, request a new password reset and change your password again. Send the `POST /forgot-password?temp-forgot-password-token` request to Burp Repeater again.

#### 6. In Burp Repeater, delete the value of the `temp-forgot-password-token` parameter in both the URL and request body. Change the `username` parameter to `carlos`. Set the new password to whatever you want and send the request.

#### 7. In the browser, log in to Carlos's account using the new password you just set. Click `My account` to solve the lab.

<hr/>