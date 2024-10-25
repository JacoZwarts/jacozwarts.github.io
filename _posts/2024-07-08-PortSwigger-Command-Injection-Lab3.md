---
layout: post
title:  "PortSwigger - Command Injection - Lab 3"
date:   2024-07-08 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [command-injection,bscp]
categories: [PortSwigger]
---

# Lab 3 - Blind OS command injection with output redirection
><b>Lab Objective:</b>
<br/><br/>
This lab contains a blind OS command injection vulnerability in the feedback function.
The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response. However, you can use output redirection to capture the output from the command. There is a writable folder at:
/var/www/images/
The application serves the images for the product catalog from this location. You can redirect the output from the injected command to a file in this folder, and then use the image loading URL to retrieve the contents of the file.
To solve the lab, execute the whoami command and retrieve the output.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection">Port Swigger - Lab 3</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/38308244">Rana Khalil - Command Injection -  Lab3</a>
<br/>



### Solution
<hr/>

#### 1. Include images in Intruder's http filters:
![Include images in http filter](/images/portswigger/CommandInjection/lab3/command_injection_lab_3_1.png)


#### 2. Notice the request url for images: `/image?filename=72.jpg`
![Command Injection images](/images/portswigger/CommandInjection/lab3/command_injection_lab_3_2.png)

#### 3. We'll test all user input fields for command injection by using:
`& sleep 10 #`

#### 4. By testing all possible input fields we've identified that the submit feedback email parameter is vulnerable to OS command injection.
![Request parameter vulnerable to command injection identified](/images/portswigger/CommandInjection/lab3/command_injection_lab_3_3.png)

Notice the server is taking longer to respond to our request this is because we've successfully exploited the OS command injection vulnerability.
![Command injection server response time](/images/portswigger/CommandInjection/lab3/command_injection_lab_3_4.png)

#### 5. Seeing that we're dealing with blind command injection we'll try to create a file at a location we can access via the web application and save the whoami output to that file.

We'll write the output of whoami to `/var/www/images/output.txt`
```
& whoami > var/www/images/output.txt #
```

#### 6. We'll turn on Interceptor and intercept the submit feedback request and add the above payload to the email parameter and URL encode the payload:

![Command injection payload](/images/portswigger/CommandInjection/lab3/command_injection_lab_3_5.png)

#### 7. We'll open the file we've just created by navigating to:
```
/image?filename=output.txt
```
![Whoami result in output file](/images/portswigger/CommandInjection/lab3/command_injection_lab_3_6.png)
<hr/>