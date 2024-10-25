---
layout: post
title:  "PortSwigger - Authentication - Lab 7"
date:   2024-06-04 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,bscp]
categories: [PortSwigger]
---

# Lab 7 - Authentication - Username enumeration via account lock
><b>Lab Objective:</b>
<br/><br/>
This lab is vulnerable to username enumeration. It uses account locking, but this contains a logic flaw. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
<br/>
<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-usernames">Candidate usernames</a>
<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">Candidate passwords</a>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-account-lock">Port Swigger - Lab 7</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, investigate the login page and submit an invalid username and password. Send the `POST /login` request to Burp Intruder.

#### 2. Select the attack type `Cluster bomb`. Add a payload position to the `username` parameter. Add a blank payload position to the end of the request body by clicking Add § twice. The result should look something like this: `username=§invalid-username§&password=example§§`

#### 3. On the `Payloads` tab, add the list of usernames to the first payload set. For the second set, select the `Null payloads` type and choose the option to generate 5 payloads. This will effectively cause each username to be repeated 5 times. Start the attack.

#### 4. In the results, notice that the responses for one of the usernames were longer than responses when using other usernames. Study the response more closely and notice that it contains a different error message: `You have made too many incorrect login attempts.` Make a note of this username.

#### 5. Create a new Burp Intruder attack on the POST `/login request`, but this time select the Sniper attack type. Set the `username` parameter to the username that you just identified and add a payload position to the `password` parameter.

#### 6. Add the list of passwords to the payload set and create a grep extraction rule for the error message. Start the attack.

#### 7. In the results, look at the grep extract column. Notice that there are a couple of different error messages, but one of the responses did not contain any error message. Make a note of this password.

#### 8. Wait for a minute to allow the account lock to reset. Log in using the username and password that you identified and access the user account page to solve the lab.

<hr/>