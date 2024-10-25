---
layout: post
title:  "PortSwigger - Directory Traversal - Lab 2"
date:   2024-06-10 00:00
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal,bscp]
categories: [PortSwigger]
---

# Lab 2 - Directory Traversal - File path traversal, traversal sequences blocked with absolute path bypass
><b>Lab Objective:</b>
<br/><br/>
This lab contains a path traversal vulnerability in the display of product images.
The application blocks traversal sequences but treats the supplied filename as being relative to a default working directory.<br/><br/>
To solve the lab, retrieve the contents of the `/etc/passwd` file.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/file-path-traversal/lab-absolute-path-bypass">Port Swigger - Lab 2</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44691836">Rana Khalil - Lab 2</a>
<br/>

### Solution
<hr/>

#### 1. Notice all the image requests when loading the lab. 

![Directory Traversal - Lab 2 - Image Response from server](/images/portswigger/DirectoryTraversal/lab2/directory-traversal-lab2-image-requests.png)


#### 2. Send one of these requests to Repeater and access the file by using the absolute path: `/etc/passwd`

![Directory Traversal - Lab 2 - Payload](/images/portswigger/DirectoryTraversal/lab2/directory-traversal-lab2-payload.png)

### Absolute vs. Relative Paths
- `Relative Path:` Specifies a file or directory location relative to the current working directory. It does not start from the root directory and often includes navigation symbols like `.` (current directory) or `..` (parent directory).
  - Examples:
    - `documents/file.txt` (if the current directory is `/home/user`)
    - `../file.txt` (refers to the parent directory's file)
- `Absolute Path:` Always starts from the root and does not depend on the current working directory.
  - Examples:
    - `/home/user/documents/file.txt`
    - `C:\Users\Username\Documents\file.txt`

<hr/>