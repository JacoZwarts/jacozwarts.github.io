---
layout: post
title:  "PortSwigger - Authentication - Lab 1"
date:   2024-06-03 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [command-injection]
categories: [PortSwigger]
---

# Lab 1 - Authentication - Username enumeration via different responses
><b>Lab Objective:</b>
<br/><br/>
This lab is vulnerable to username enumeration and password brute-force attacks. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.
<br/>
The lab has an account with a predictable username and password, which can be found in the following wordlists:
<br/>
<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-usernames">Candidate usernames</a>
<br/>
<a href="https://portswigger.net/web-security/authentication/auth-lab-passwords">Candidate passwords</a>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses">Port Swigger - Lab 1</a>
<br/>




### Solution
<hr/>

#### 1. With Burp running, investigate the login page and submit an invalid username and password.


#### 2. In Burp, go to Proxy > HTTP history and find the `POST /login` request. Highlight the value of the `username` parameter in the request and send it to Burp Intruder.

#### 3. In Burp Intruder, go to the Positions tab. Notice that the `username` parameter is automatically set as a payload position. This position is indicated by two `§` symbols, for example: `username=§invalid-username§`. Leave the password as any static value for now.

#### 4. Make sure that the Sniper attack type is selected.

#### 5. On the `Payloads` tab, make sure that the `Simple list` payload type is selected.

#### 6. Under `Payload settings`, paste the list of candidate usernames. Finally, click Start attack. The attack will start in a new window.

#### 7. When the attack is finished, on the `Results` tab, examine the `Length` column. You can click on the column header to sort the results. Notice that one of the entries is longer than the others. Compare the response to this payload with the other responses. Notice that other responses contain the message `Invalid username`, but this response says `Incorrect password`. Make a note of the username in the `Payload` column.

#### 8. Close the attack and go back to the `Positions` tab. Click `Clear`, then change the `username` parameter to the username you just identified. Add a payload position to the `password` parameter. The result should look something like this: `username=identified-user&password=§invalid-password§`

#### 9. On the `Payloads` tab, clear the list of usernames and replace it with the list of candidate passwords. Click `Start attack`.

#### 10. When the attack is finished, look at the `Status` column. Notice that each request received a response with a `200` status code except for one, which got a `302` response. This suggests that the login attempt was successful - make a note of the password in the `Payload` column.

#### 11. Log in using the username and password that you identified and access the user account page to solve the lab.



<hr/>