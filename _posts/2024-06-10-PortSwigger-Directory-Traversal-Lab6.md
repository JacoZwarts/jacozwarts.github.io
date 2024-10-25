---
layout: post
title:  "PortSwigger - Directory Traversal - Lab 6"
date:   2024-06-10 00:00
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal]
categories: [PortSwigger]
---

# Lab 6 - Directory Traversal - File path traversal, validation of file extension with null byte bypass
><b>Lab Objective:</b>
<br/><br/>
This lab contains a path traversal vulnerability in the display of product images.
The application validates that the supplied filename ends with the expected file extension.
<br/><br/>
To solve the lab, retrieve the contents of the `/etc/passwd` file.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/file-path-traversal/lab-validate-file-extension-null-byte-bypass">Port Swigger - Lab 6</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44691844">Rana Khalil - Lab 5</a>
<br/>

### Solution
<hr/>

Null byte injection, also known as null termination, can be used to bypass certain security checks. In C-based languages, strings are terminated with a null byte (`\0`). This means that anything after the null byte is ignored. Attackers can exploit this behavior to manipulate file paths.

#### 1. Append null byte `%00` before filename in our below example append the null byte before 123.png

```
/image?filename=../../../etc/passwd%00123.png
```

![Directory Traversal - Lab 6 - null byte payload](/images/portswigger/DirectoryTraversal/lab6/directory-traversal-lab6.png)

<hr/>