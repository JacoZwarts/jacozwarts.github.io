---
layout: post
title:  "PortSwigger - Directory Traversal - Lab 1"
date:   2024-06-10 00:00
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal]
categories: [PortSwigger]
---

# Lab 1 - Directory Traversal - Simple Case
><b>Lab Objective:</b>
<br/><br/>
This lab contains a path traversal vulnerability in the display of product images.<br/><br/>
To solve the lab, retrieve the contents of the `/etc/passwd` file.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/file-path-traversal/lab-simple">Port Swigger - Lab 1</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44691834">Rana Khalil - Lab 1</a>
<br/>

### Solution
<hr/>

#### 1. Investigate http requests and notice the `/image?filename` request
![Directory Traversal - Lab 1 - Image Response from server](/images/portswigger/DirectoryTraversal/lab1/directory-traversal-lab1-image-response.png)


#### 2. We'll send the request to intruder and remove the image.jpg and replace it with the below payload  to read the etc/passwd file.

```
../../../../../../../../etc/passwd
```
![Directory Traversal - Lab 1 - Image Request](/images/portswigger/DirectoryTraversal/lab1/directory-traversal-lab1-image-request.png)

<hr/>