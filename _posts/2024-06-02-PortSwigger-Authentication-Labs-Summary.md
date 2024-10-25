---
layout: post
title:  "PortSwigger - Authentication - Labs Summary"
date:   2024-06-02 20:40
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,bscp]
categories: [PortSwigger]
---

## Lab 1 - Username and password enumeration:
Basic brute force attack use intruder and enumerate the username until we found an error message identifying the password is incorrect. Once we've identified the username we'll enumerate the password until we get a 302 redirect response.
## Lab 2 - 2FA bypass:
This lab uses a flawed 2FA feature, login with your account and enter the 2FA code, notice that we're redirect to `/my-account` Log into the victims account and manually navigate to the `/my-account` page.
## Lab 3 - Password reset broken logic:
Investigate the forgot my password functionality with your own details, notice that this feature submits the username as a hidden field. Removing the temp-forgot-password-token in both the url parameter and the body still allows us to change our password. Change the username value to Carlos and change Carlos's password.
## Lab 4 - Username enumeration via subtle different responses:
Enumerate usernames and mark the error message via the GREP functionality notice that one response has a different error message than the others (A trailing space). Now enumerate the passwords and notice the request with a 302 redirect code.
## Lab 5 - Username enumeration via Response timing:
Using the `X-Forwarded-For` header with the Pitch Fork attack to mock changing IP addresses. Identify the username by looking at the requests response times, the valid username's response time is a lot longer than the invalid username requests.
## Lab 6 - Broken brute-force protection, IP Block:
Lockout logic gets reset when you login with valid credentials. Every 3rd request we'll login with the valid user's credentials. We'll use a python script to generate a password-wordlist that enters the valid user's password in every 3rd place. We'll also set the simple list in Burp Suite Intruder as Carlos,Carlos,Wiener and repeat this 50 times because we have 100 passwords in the original password-wordlist.
## Lab 7 - username enumeration via account lock:
To find the username we'll use Cluster Bomb. We'll add a payload marker to the username and also add a second payload marker. In the first payload we'll use a simple list and load the usernames. For the second payload we'll use Null Payloads 5 times. The correct username has a longer response time than the other requests.
## Lab 8 - 2FA broken Logic
Identify the verify parameter in the 2FA request payload, change this to Carlos and send the request. Login with your credentials and change the verify parameter to Carlos to generate a valid 2FA code for Carlos. Go back to the request that submits the 2FA code and replace the verify parameter with carlos and brute force the 2FA code. Notice the 302 redirect request.
## Lab 9 - Brute-forcing a stay-logged in cookie
Notice a stay logged in cookie is saved in the browser. Take this cookie and Base 64 decode it and notice it's username:MD5 Hash. Notice that this is your username and your password's MD5 hash. Use burp intruder to load all of the passwords and hash the password append Carlos: to the MD5 hashed password and base65 encrypt the string and send it to the backend. Notice the 302 redirect request.
## Lab 10 - Offline password cracking
Use XSS to steal the user's cookie and use an online MD5 Hash tool to retrieve the password
## Lab 11 - Password reset poisoning via middleware
We've added the `X-Forwarded-Host` header to the request to inform the backend server that the `Host` header was overwritten. The correct host value it should use is now stored in `X-Forwarded-Host`. The lab uses the `Host` header, appends the forgot password URL and query parameter, and then sends the reset password URL to the user to reset their password. We tricked the lab into using our malicious URL and sending it to Carlos. Carlos clicked on the link in his email, which forwarded the request to our malicious HTTP server with a valid reset token. We extracted this reset token and then changed Carlos's password.
<br/>
<br/>
`Note: When using X-Forwarded-Host ensure you don't include the http/https`

## Lab 12 - Password brute-force via password change
When changing your password notice the behavior when you enter the wrong current password. If the two entries for the new password match, the account is locked. However, if you enter two different new passwords, an error message simply states `Current password is incorrect`. If you enter a valid current password, but two different new passwords, the message says `New passwords do not match`. We can use this message to enumerate correct passwords
## Lab 13 - Broken brute-force protection, multiple credentials per request
Notice the payload to login is JSON change the password from string to an array of strings and add all the passwords from the password-wordlist into this string array.
## Lab 14 - 2FA bypass using a brute-force attack
When logging in with the credentials we're prompted to supply a 2FA code. When trying to brute force the 2FA code we get logged out after two attempts, we'll use BurpSuite's macro functionality to automatically log us back in and then continue brute forcing the 2FA code.
<hr/>