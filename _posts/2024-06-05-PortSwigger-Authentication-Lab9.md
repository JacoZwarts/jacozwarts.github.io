---
layout: post
title:  "PortSwigger - Authentication - Lab 9"
date:   2024-06-05 18:30
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication]
categories: [PortSwigger]
---

# Lab 9 - Brute-forcing a stay-logged-in cookie
><b>Lab Objective:</b>
<br/><br/>
This lab allows users to stay logged in even after they close their browser session. The cookie used to provide this functionality is vulnerable to brute-forcing.<br/>
To solve the lab, brute-force Carlos's cookie to gain access to his "My account" page.
<br/>
<br/>
`Your credentials: wiener:peter`<br/>
`Victim's username: carlos`
<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">Candidate passwords</a>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie">Port Swigger - Lab 9</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, log in to your own account with the `Stay logged in` option selected. Notice that this sets a `stay-logged-in` cookie

#### 2. Examine this cookie in the Inspector panel and notice that it is Base64-encoded. Its decoded value is `wiener:51dc30ddc473d43a6011e9ebba6ca770`. Study the length and character set of this string and notice that it could be an MD5 hash. Given that the plaintext is your username, you can make an educated guess that this may be a hash of your password. Hash your password using MD5 to confirm that this is the case. We now know that the cookie is constructed as follows: `base64(username+':'+md5HashOfPassword)`

#### 3. Log out of your account.

#### 4. In the most recent `GET /my-account`, highlight the `stay-logged-in` cookie parameter and send the request to Burp Intruder.

#### 5. In Burp Intruder, notice that the `stay-logged-in` cookie has been automatically added as a payload position. Add your own password as a single payload.

#### 6. Under Payload processing, add the following rules in order. These rules will be applied sequentially to each payload before the request is submitted.

##### - Hash: `MD5`
##### - Add prefix: `wiener:`
##### - Encode: `Base64-encode`

#### 7. As the Update email button is only displayed when you access the `/my-account` page in an authenticated state, we can use the presence or absence of this button to determine whether we've successfully brute-forced the cookie. On the Settings tab, add a grep match rule to flag any responses containing the string `Update email`. Start the attack

#### 8. Notice that the generated payload was used to successfully load your own account page. This confirms that the payload processing rules work as expected and you were able to construct a valid cookie for your own account.

#### 9. Make the following adjustments and then repeat this attack:
##### - Remove your own password from the payload list and add the list of <a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">candidate passwords</a> instead.
##### - Change the Add `prefix` rule to add `carlos:` instead of `wiener:`
<hr/>