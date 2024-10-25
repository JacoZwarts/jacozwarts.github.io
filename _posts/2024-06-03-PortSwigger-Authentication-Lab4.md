---
layout: post
title:  "PortSwigger - Authentication - Lab 4"
date:   2024-06-03 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,bscp]
categories: [PortSwigger]
---

# Lab 4 - Authentication - Username enumeration via subtle different responses
><b>Lab Objective:</b>
<br/><br/>
This lab is subtly vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:
<br/>
<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-usernames">Candidate usernames</a>
<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">Candidate passwords</a>
<br/>
<br/>
To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses">Port Swigger - Lab 4</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, submit an invalid username and password. Highlight the `username` parameter in the `POST /login` request and send it to Burp Intruder.

#### 2. On the `Payloads` tab, notice that the `username` parameter is automatically marked as a payload position. Make sure that the Simple list payload type is selected and add the list of candidate usernames.

#### 3. On the `Settings` tab, under `Grep - Extract`, click `Add`. In the dialog that appears, scroll down through the response until you find the error message `Invalid username or password.`. Use the mouse to highlight the text content of the message. The other settings will be automatically adjusted. Click `OK` and then start the attack.

#### 4. When the attack is finished, notice that there is an additional column containing the error message you extracted. Sort the results using this column to notice that one of them is subtly different.

#### 5. Look closer at this response and notice that it contains a typo in the error message - instead of a full stop/period, there is a trailing space. Make a note of this username.

#### 6. Close the attack and go back to the `Positions` tab. Insert the username you just identified and add a payload position to the `password` parameter: `username=identified-user&password=§invalid-password§`

#### 7. On the `Payloads` tab, clear the list of usernames and replace it with the list of passwords. Start the attack.

#### 8. When the attack is finished, notice that one of the requests received a `302` response. Make a note of this password.

#### 9. Log in using the username and password that you identified and access the user account page to solve the lab

<hr/>