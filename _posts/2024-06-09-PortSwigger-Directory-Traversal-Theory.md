---
layout: post
title:  "PortSwigger - Directory Traversal - Theory"
date:   2024-06:09 13:40
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal]
categories: [PortSwigger]
---

# Introduction
>Directory traversal or also known as file path traversal is a vulnerability that allows an attacker to read files on the server that is running the application.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44691833">Rana Khalil Academy - Directory Traversal</a>
<br/>
<a href="https://portswigger.net/web-security/file-path-traversal">PortSwigger - Path traversal</a>
<br/>
<a href="https://owasp.org/www-community/attacks/Path_Traversal">OWASP - Path traversal</a>
<br/>

### Impact of Directory Traversal Vulnerabilities:
<hr/>

- Unauthorized access to the application
  - `Confidentiality` - Allows you to read files on the system
  - `Integrity` - Some cases allow you to run commands and therefore alter files on the system.
  - `Availability` - Some cases allow you to run commands and therefore delete files on the system.
- If the directory traversal vulnerability allows you to run commands, then you can get full execution on the server.

### Finding directory traversal Vulnerabilities:
<hr/>

#### Black box testing:

- Map application
 - Identify all instances where the web application appears to contain the name of a file or directory.
 - Identify all functions in the applications whose implementation is likely t involve retrieval of data from server filesystem.
- Test identified instances with common directory traversal payloads and observe how the application responds.
- Automate testing using a web application vulnerability scanner (WAVS)

#### White box testing:

- Identify instances where user-supplied input is being passed to file APIs or as parameters to the operating system.
  - Identify instances in a running application first (black-box perspective) and then review the code responsible for that functionality.
  - Grep on functions in the code that are known to include and evaluate files on the server and review if they are user supplied input.
  - Use a tool to monitor all filesystem activity on the server. Then test each page of the application by inserting a single unique string. Set a filter in your monitoring tool for that specific string and identify all filesystem events that contain the string.
- Validate potential directory traversal vulnerabilities on a running application.

### Exploiting directory traversal vulnerabilities:

#### Regular case

```
../../../../../../etc/passwd
.\..\..\..\..\..\windows\win.ini
```

#### Absolute paths

```
/etc/passwd
```

#### Traversal sequences stripped non-recursively:

```
.. ../ /.. ../ /.. ../ / etc/passwd
```
Leaving us with: `../../../etc/passwd`

- Bypass traversal sequence stripped defense using URL encoding
- Bypass start of path validation<br/>
`/var/www/images/../../../etc/passwd`
- Bypass file extension validation using null byte<br/>
`../../../etcpasswd%00.png`

### Preventing directory traversal:

- Avoid passing user-supplied input to filesystem APIs
- Validate input compared to allow list of permitted values
- Use filesystem APIs to canonicalize the path and verify that it starts with the expected base directory.

<hr/>