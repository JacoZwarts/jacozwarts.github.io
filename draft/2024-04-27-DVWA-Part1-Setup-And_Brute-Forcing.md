---
layout: post
title:  "Damn Vulnerable Web Application - Part 1 / 12 Setup & Brute Force"
date:   2024-04-27 01:07
image:  /images/dvwa-walkthrough/DVWA.jpg
tags:   [brute-force]
categories: [DVWA]
---

# Introduction
>Damn Vulnerable Web App (DVWA) is a PHP/MySQL web application that is damn vulnerable. Its main goals are to be an aid for security professionals to test their skills and tools in a legal environment, help web developers better understand the processes of securing web applications and aid teachers/students to teach/learn web application security in a class room environment.
<br/>
<br/>
<b>Reference:</b>
<a href="https://www.vulnhub.com/entry/damn-vulnerable-web-application-dvwa-107,43/">vulnhub</a>

### Follow the [setup](https://github.com/digininja/DVWA?tab=readme-ov-file#docker) instruction. 

## Running DVWA for the first time: 
1. Navigate to your DVWA directory and run the following command:
<br/>
`sudo docker compose up`
<br/>

![DVWA docker containers running](/images//dvwa-walkthrough/docker-running.png)
<br/>

2. Navigate to `http://localhost:4280`
<br/>

![DVWA login page](/images//dvwa-walkthrough/dvwa-login-page.png)
<br/>

3. Use the following credentials:
<br/> `Username: admin`
<br/> `Password: password`
<br/>

![DVWA home page](/images//dvwa-walkthrough/dvwa-home-page.png)
<br/>

4. Set DVWA Security level to `Low`.
<br/>

![DVWA security level](/images//dvwa-walkthrough/dvwa-security-level.png)
<br/>

## Brute Force
>A brute force attack can manifest itself in many different ways, but primarily consists in an attacker configuring predetermined values, making requests to a server using those values, and then analyzing the response. For the sake of efficiency, an attacker may use a dictionary attack (with or without mutations) or a traditional brute-force attack (with given classes of characters e.g.: alphanumeric, special, case (in)sensitive). Considering a given method, number of tries, efficiency of the system which conducts the attack, and estimated efficiency of the system which is attacked the attacker is able to calculate approximately how long it will take to submit all chosen predetermined values.
<br/><br/>
<b>Reference:</b>
<a href="https://owasp.org/www-community/attacks/Brute_force_attack">OWASP</a>

### Security Level: `Low` & `Medium`

![DVWA Bruteforce](/images//dvwa-walkthrough/bruteforce/bruteforce-low.png)
<br/>

1. For the intial request we'll use the below credentials:
<br/>
<br/> `Username: admin`
<br/> `Passowrd: admin`
<br/>

2. Capture the request in Burp and send it to intruder (ctrl + I):
<br/><br/>

![DVWA Bruteforce intial request](/images//dvwa-walkthrough/bruteforce/bruteforce-low-intial-request.png)
<br/>

3. Insert a payload marker on the password query parameter:
<br/>

![DVWA Bruteforce payload marker](/images//dvwa-walkthrough/bruteforce/bruteforce-intuder-setup.png)
<br/>
4. Configure the payload and use the /usr/share/wordlists/john.lst as the wordlist
<br/>

![DVWA Bruteforce payload configuration](/images//dvwa-walkthrough/bruteforce/bruteforce-payload-configuration.png)
<br/>
5. Filter by Length in descending order and click on the response tab and search for "Welcome"
<br/>

![DVWA Bruteforce result](/images//dvwa-walkthrough/bruteforce/bruteforce-result.png)

#### Source Code:

![DVWA Bruteforce Source Code Low](/images//dvwa-walkthrough/bruteforce/bruteforce-low-source-code.png)
<br/>

![DVWA Bruteforce Source Code Medium](/images//dvwa-walkthrough/bruteforce/bruteforce-medium-source-code.png)
<br/>
### Security Level: `High`

1. Capture the login request in Burp and notice the extra query parameter called user_token. This is a <a href="https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#token-based-mitigation">CSRF Token</a> generated with every request.
2. Send the request to Burp Intruder and select `Pitchfork` as the attack type, and add two payoad markers one for password and another for user_token.
<br/>

![DVWA Bruteforce High Intruder Configuration](/images//dvwa-walkthrough/bruteforce/bruteforce-high-intruder-setup.png)
3. For the first payload load the john.lst for the passwords we'll be iterating through. For payload 2 set the payload type as Recursive Grep.

![DVWA Bruteforce High - Configure Payload settings](/images/dvwa-walkthrough/bruteforce/bruteforce-high-configure-payload.png)
<br/>

4. Click on Settings and add a new Grep Extract item.

<br/>

![DVWA Bruteforce High - Configure Grep settings](/images/dvwa-walkthrough/bruteforce/bruteforce-high-configure-grep.png)
<br/>

5. Change the redirect configuration to `Always`

![DVWA Bruteforce High - Redirect configuration](/images/dvwa-walkthrough/bruteforce/bruteforce-high-redirect-configuration.png)
<br/>

6. Configure a new pool with only one thread to ensure only send one request at a time to use valid CSRF tokens with every request.

![DVWA Bruteforce High - Resource pool configuration](/images//dvwa-walkthrough/bruteforce/brute-force-high-resource-pool.png)
<br/>

7. We've bypassed the CSRF token and found the result: 

![DVWA Bruteforce High result](/images//dvwa-walkthrough/bruteforce/bruteforce-high-result.png)

#### Source Code:

![DVWA Bruteforce Source Code High](/images//dvwa-walkthrough/bruteforce/bruteforce-high-source-code.png)
<br/>

