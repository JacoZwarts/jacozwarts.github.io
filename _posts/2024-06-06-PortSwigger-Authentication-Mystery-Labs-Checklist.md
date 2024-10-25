---
layout: post
title:  "PortSwigger - Authentication - Mystery Labs Checklist"
date:   2024-06-06 20:40
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,checklist,bscp]
categories: [PortSwigger]
---

# Authentication Mystery Labs Checklist
>There are two types of authentication methods we will examine. The first involves using credentials, where we will test default credentials. The second involves no credentials, where we will employ brute-force techniques to gain access to the system, either by finding a new username or by brute-forcing Carlos's account.



## Labs with credentials
><br/><b>Credentials to try:</b>
<br/>
wiener:peter<br/>
carlos:montoya<br/><br/>

### 2FA: `(Carlos)`
- Login with credentials and change url to /my-account 
- Look at 2FA payload check if the payload contains a token and  an username. 
- Submit invalid 2FA codes and check if you get logged out, if so you'll have to use a macro to log you in with your valid credentials.

### Password Reset: `(Carlos)`
- Check if payload contains a token and username. (Remove token and change username).
- When changing password check if you receive an email with the token. (Middleware attack using X-Forwarded-Host header)
- When password ask for new-password-1 and new-password-2 ensure the values don't match and enumerate the password-wordlist.

### Stay logged in Cookie: `(Carlos)`
Check if a stay-logged-in cookie is added.
- Check if the post functionality is vulnerable to XSS.
- Get the MD5 hash and use an online decoder to find the password.
- Check if the cookie is base64 encrypted. Iterate through the password-wordlist and generate the MD5 hash of each value. Prefix Carlos: with each of the MD5 hashes and then base 64 encode the new string.

### Account lockout:
Check for account lockout.
- Test if X-Forwarded-For header is allowed and use a password with minimum of 100 characters to see a difference in response timing. (Use pitchfork to change value of the header and enumerate passwords) `(Not Carlos)`
- Create a macro to log into your account and then iterate Carlos's account twice repeat this until you find a 302 redirect response. `(Carlos)`

## Labs without credentials
- Brute-force check error message for differences.
- Brute-force when supplying a long password check if response times are different.
- Brute-force with null payloads to get locked out on valid username.
- Check if login payload is Json, if it is add string array of passwords.
<hr/>