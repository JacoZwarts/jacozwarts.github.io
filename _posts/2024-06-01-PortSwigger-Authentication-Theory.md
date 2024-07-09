---
layout: post
title:  "PortSwigger - Authentication - Theory"
date:   2024-06-01 20:40
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication]
categories: [PortSwigger]
---

# Introduction
>Authentication is the process of identifying a user or a client. There is three types of authentication types:

- Knowledge Factor: Something you know such as a password
- Possession Factor: Something you have such as a mobile device or a security token.
- Inherence Factor: Something you are or do. Bio metrics or a patterns of behavior.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://academy.ranakhalil.com/courses/web-security-academy-video-series/lectures/45273752">Rana Khalil Academy - Authentication</a>
<br/>
<a href="https://portswigger.net/web-security/authentication">PortSwigger - Authentication vulnerabilities</a>
<br/>
<a href="https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html">OWASP - Authentication Cheat Sheet</a>


### What's the difference between authentication and authorization?
<hr/>
Authentication is the process of verifying a user is who they claim to be, authorization is verifying if a user is allowed to do something.

### Authentication Vulnerabilities:
<hr/>

- Weak password requirements
- Improper restriction of authentication attempts
- Verbose error message
- Vulnerable transmission of credentials
- Insecure Forgot Password Functionality
- Defects in Multistage login mechanism
- Insecure Storage of Credentials

#### Weak Password Requirement

<a href="https://danielmiessler.com/p/casmm-consumer-authentication-security-maturity-model">CASMM module should be used.</a>

- Very short or blank
- Common dictionary words or names
- Password is the same as the username
- Use of default password
- Missing or ineffective MFA
<br/>

#### Improper Restriction of Authentication Attempts

Application permits brute force or other automated attacks.

- Login Page
- OTP / MFA Page
- Change password Page
<br/>

#### Verbose Error Message

The application outputs a verbose error message that allows for username enumeration.

- Error message should not specify which parameter is incorrect. Should be a generic error message.

#### Vulnerable Transmission of Credentials

The application uses an un-encrypted HTTP connection to transmit login credentials

#### Insecure Forgot Password Functionality

Design weaknesses in the forgotten password functionality usually makes the weakest link that can be used to attack the application's overall authentication logic.

#### Defects in Multistage Login Mechanism

Insecure implementation of the MFA functionality. Passing the user's credentials as parameters.

#### Insecure Storage of Credentials

Uses plain text, encrypted, or weekly hashed password data stores.
This is both a cryptographic failure vulnerability and an authentication vulnerability.
Passwords should be hashed and not encrypted.

![Authentication password storage](/images/portswigger/Authentication//authentication-theory-1.png)

### Unauthorized access to the application leads to:
<hr/>

- `Confidentiality` - Access to other users' data.
- `Integrity` - Access to update other users' data.
- `Availability` - Access to delete users and their data.

### Find and Exploit

#### Weak Password Complexity Requirements:

- Review the website for any description of the rules.
- If self registration is possible, attempt to register several accounts with different kinds of weak passwords to discover what rules are in place:
  - Very short or blank.
  - Common dictionary words or names.
  - Password is the same as the username.
- If you control a single account and password change is possible, attempt to change the password to various weak values.

#### Improper Restriction of Authentication Attempts:

 - Manually submit several bad login attempts for an account control.
 - After 10 failed login attempts, if the application does not return a message about account lockout, attempt to log in correctly. If it works, then there is no lockout mechanism.
   - Run a brute force attack to enumerate the valid password. 
     - Use tools like Hydra, Burp Intruder.
- If the account is locked out, monitor the requests and responses to determine if the lockout mechanism is insecure

#### Verbose Error Message:

- Submit a request with a valid username and an invalid password.
- Submit a request with an invalid username.
- Review both responses for any differences in the status code, any redirects, information displayed on the screen, HTML page source, or even the time to process the request.
- If there is a difference, run a brute force attack to enumerate the list of valid usernames in the application.

#### Vulnerable Transmission of Credentials:

- Perform a successful login while monitoring all traffic in both directions between the client and server.
- Look for instances where credentials are submitted in a URL query string or a cookie, or are transmitted back from the server to the client.
- Attempt to access the application over HTTP and if there are any redirects to HTTPS.

#### Insecure Forgot Password Functionality:

 - Identify if the application has any forgotten password functionality
 - If it does, perform a complete walk-through of the forgot password functionality using an account you have control of while intercepting the requests / responses in a proxy.
 - Review the functionality to determine if it allows for username enumeration or brute-force attacks.
 - If the application generates an email containing a recovery URL, obtain a number of these URLs and attempt to identify any predictable patterns or sensitive information included in the URL. Also check if the URL is long lived and does not expire.
 #### Defects in Multistage login Mechanism:
 - Identify if the application uses a multistage login mechanism.
 - If it does, perform a complete walk-through using an account you have control of while intercepting the requests / responses in a proxy.
 - Review the functionality to determine if the allows for username enumeration or brute-force attacks.
 #### Insecure Storage of Credentials:
 - Review all the application's authentication related functionality. If you find any instances where the user's password is transmitted to the client plaintext or obfuscated, this indicates the passwords are being stored insecurely.
- If you gain remote code execution (RCE) on the server, review the database to determine if the passwords are stored insecurely.
- Conduct technical interviews with the developers to review how passwords are stored in the backend database

#### Reference:
<a href="https://github.com/rkhal101/Web-Security-Academy-Series/blob/main/broken-authentication/theory/Authentication%20Vulnerabilities%20Complete%20Guide%20Theory%20Video%20Slides.pdf">Rana Khalil - Authentication Vulnerabilities Complete Guide Theory</a>
<hr/>