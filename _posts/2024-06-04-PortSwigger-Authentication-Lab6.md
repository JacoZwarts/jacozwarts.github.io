---
layout: post
title:  "PortSwigger - Authentication - Lab 6"
date:   2024-06-04 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,bscp]
categories: [PortSwigger]
---

# Lab 6 - Authentication - Broken brute-force protection, IP block
><b>Lab Objective:</b>
<br/><br/>
This lab is vulnerable due to a logic flaw in its password brute-force protection. To solve the lab, brute-force the victim's password, then log in and access their account page
<br/>
<br/>
Your credentials: wiener:peter<br/>
Victim's username: carlos<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">Candidate passwords</a>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block">Port Swigger - Lab 6</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, investigate the login page. Observe that your IP is temporarily blocked if you submit 3 incorrect logins in a row. However, notice that you can reset the counter for the number of failed login attempts by logging in to your own account before this limit is reached

#### 2. Enter an invalid username and password, then send the `POST /login` request to Burp Intruder. Create a pitchfork attack with payload positions in both the `username` and `password` parameters.

#### 3. On the `Resource pool` tab, add the attack to a resource pool with `Maximum concurrent requests` set to `1`. By only sending one request at a time, you can ensure that your login attempts are sent to the server in the correct order

#### 4. On the `Payloads` tab, select payload set 1. Add a list of payloads that alternates between your username and `carlos`. Make sure that your username is first and that `carlos` is repeated at least 100 times.

#### 5. Edit the list of candidate passwords and add your own password before each one. Make sure that your password is aligned with your username in the other list.

#### 6. Add this list to payload set 2 and start the attack

#### 7. When the attack finishes, filter the results to hide responses with a `200` status code. Sort the remaining results by username. There should only be a single `302` response for requests with the username `carlos`. Make a note of the password from the `Payload 2` column.

#### 8. Log in to Carlos's account using the password that you identified and access his account page to solve the lab.

<hr/>