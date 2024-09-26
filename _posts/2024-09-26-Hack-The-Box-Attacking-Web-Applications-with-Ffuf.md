---
layout: post
title:  "Hack The Box - Academy - Attacking Web Applications with Ffuf"
date:   2024-09-26 14:36
image:  /images/htb/awa-with-ffuf/htb-awa-ffuf.png
tags:   [recon]
categories: [htbacademy]
---

><b>Overview:</b>
<br/><br/>
You are given an online academy's IP address but have no further information about their website. As the first step of conducting a Penetration Test, you are expected to locate all pages and domains linked to their IP to enumerate the IP and domains properly.<br/><br/>
Finally, you should do some fuzzing on pages you identify to see if any of them has any parameters that can be interacted with. If you do find active parameters, see if you can retrieve any data from them.<br/>
<br/>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/54/section/511">Attacking Web Applications with Ffuf</a>
<br/>

## Q1: Run a sub-domain/vhost fuzzing scan on '*.academy.htb' for the IP shown above. What are all the sub-domains you can identify? (Only write the sub-domain name)

#### Execute the below command to add academy.htb to /etc/hosts
```
sudo sh -c 'echo "{TARGET_IP} academy.htb" >> /etc/hosts'
```

#### Execute the below command to start the sub-domain/vhost scan

```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:{PORT}/ -H 'Host: FUZZ.academy.htb' -fs 985
```

![Sub-domain and vhost scan results](/images/htb/awa-with-ffuf/subdomain-vhost-scan-results.png)

### Answer: 
 `test, archive, faculty`
<hr/>

## Q2: Before you run your page fuzzing scan, you should first run an extension fuzzing scan. What are the different extensions accepted by the domains?

#### Execute the below commands to add the sub domains to /etc/hosts
```
sudo sh -c 'echo "{TARGET_IP} acrhive.academy.htb" >> /etc/hosts'
```
```
sudo sh -c 'echo "{TARGET_IP} test.academy.htb" >> /etc/hosts'
```
```
sudo sh -c 'echo "{TARGET_IP} faculty.academy.htb" >> /etc/hosts'
```

#### Execute the below command to start the extension scan for archive.academy.htb

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://archive.academy.htb:43905/indexFUZZ
```
![Extension scan result for archive.academy.htb](/images/htb/awa-with-ffuf/extension-fuzzing-archive-academy-htb.png)

#### Execute the below command to start the extension scan for test.academy.htb

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://test.academy.htb:43905/indexFUZZ
```
![Extension scan result for test.academy.htb](/images/htb/awa-with-ffuf/extension-fuzzing-test-academy-htb.png)

#### Execute the below command to faculty the extension scan for test.academy.htb

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://faculty.academy.htb:43905/indexFUZZ
```
![Extension scan result for faculty.academy.htb](/images/htb/awa-with-ffuf/extension-fuzzing-faculty-academy-htb.png)

### Answer: 
 `php, phps, php7`
<hr/>

## Q3: One of the pages you will identify should say 'You don't have access!'. What is the full page URL?

#### Execute the following command to start the directory fuzzing

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://faculty.academy.htb:PORT/FUZZ -recursion -recursion-depth 1 -e .php,.php7,.phps -fs 287 -t 200
```

![Directory Fuzzing - Result 1/3](/images/htb/awa-with-ffuf/q3-result1.png)

![Directory Fuzzing - Result 2/3](/images/htb/awa-with-ffuf/q3-result2.png)

![Directory Fuzzing - Result 3/3](/images/htb/awa-with-ffuf/q3-result3.png)

### Answer: 
 `http://faculty.academy.htb:PORT/courses/linux-security.php7`
<hr/>

## Q4: In the page from the previous question, you should be able to find multiple parameters that are accepted by the page. What are they?

#### Execute the following command to start the parameter fuzzing using GET

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:{PORT}/courses/linux-security.php7?FUZZ=key -fs 774
```
![Parameter Fuzzing - Result 1/](/images/htb/awa-with-ffuf/q4-result1.png)

![Parameter Fuzzing - Result 2/3](/images/htb/awa-with-ffuf/q4-result2.png)

#### Execute the following command to start the parameter fuzzing using POST

```
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:{PORT}/courses/linux-security.php7 -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs 774
```
![Parameter Fuzzing - Result 3/3](/images/htb/awa-with-ffuf/q4-result3.png)

### Answer: 
 `user,username`
<hr/>

## Q5: Try fuzzing the parameters you identified for working values. One of them should return a flag. What is the content of the flag?

#### Execute the following command to enumerate usernames:

```
ffuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt:FUZZ -u http://faculty.academy.htb:{PORT}/courses/linux-security.php7 -X POST -d 'username=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 781
```
![Username Enumeration - Result 1/2](/images/htb/awa-with-ffuf/q5-result1.png)

#### Execute the following command retrieve the flag:

```
curl http://faculty.academy.htb:{PORT}/courses/linux-security.php7 -X POST -d 'username=harry' -H 'Content-Type: application/x-www-form-urlencoded'

```
![Username Enumeration - Result 2/2](/images/htb/awa-with-ffuf/q5-result1.png)

### Answer: 
 `HTB{w3b_fuzz1n6_m4573r}`
<hr/>

