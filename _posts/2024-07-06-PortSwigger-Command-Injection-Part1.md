---
layout: post
title:  "PortSwigger - Command injection Part 1"
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
<a href="https://portswigger.net/web-security/os-command-injection">PortSwigger - OS Command Injection</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/38308201">Rana Khalil Academy - OS Command injection</a>

### Two conditions for OS Command Injection vulnerabilities:
1. Uses a function that can execute system commands such as exec.
1. The parameter of executable methods are user controllable.

![Code vulnerable to command injection](/images/portswigger/CommandInjection/VulnerableCommandInjectionCodeExample.png)

### Types of Command Injection

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