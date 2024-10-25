---
layout: post
title:  "PortSwigger - Authentication - Lab 11"
date:   2024-06-05 18:50
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication]
categories: [PortSwigger]
---

# Lab 11 - Password reset poisoning via middleware
><b>Lab Objective:</b>
<br/><br/>
This lab is vulnerable to password reset poisoning. The user carlos will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account. You can log in to your own account using the following credentials: wiener:peter. Any emails sent to this account can be read via the email client on the exploit server
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware">Port Swigger - Lab 11</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, investigate the password reset functionality. Observe that a link containing a unique reset token is sent via email.

#### 2. Send the `POST /forgot-password` request to Burp Repeater. Notice that the `X-Forwarded-Host` header is supported and you can use it to point the dynamically generated reset link to an arbitrary domain.

#### 3. Go to the exploit server and make a note of your exploit server URL.

#### 4. Go back to the request in Burp Repeater and add the `X-Forwarded-Host` header with your exploit server URL:  `Don't include Http/Https` `X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net`

#### 5. Change the `username` parameter to `carlos` and send the request.

#### 6. Go to the exploit server and open the access log. You should see a `GET /forgot-password` request, which contains the victim's token as a query parameter. Make a note of this token.

#### 7. Go back to your email client and copy the valid password reset link (not the one that points to the exploit server). Paste this into the browser and change the value of the `temp-forgot-password-token` parameter to the value that you stole from the victim.

#### 8. Load this URL and set a new password for Carlos's account.

#### 9. Log in to Carlos's account using the new password to solve the lab.
<hr/>