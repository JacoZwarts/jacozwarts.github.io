---
layout: post
title:  "BugForge - Daily - Cafe Club"
date:   2026-01-18 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [path-traversal,lfi]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club
><br/><b>Vulnerabilities Covered:</b>
<br/>
Path Traversal
<br/>
Local File Inclusion (LFI)
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge exploits a **path traversal vulnerability** in the application's image retrieval functionality where the server fails to sanitize user-supplied file path parameters. By intercepting a GET request that fetches images and manipulating the path parameter with directory traversal sequences (../), an attacker can navigate outside the intended directory structure to access sensitive files on the server, including the flag file.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis and Request Monitoring

Begin by exploring the CafeClub application with Burp Suite configured to intercept traffic. Disable Burp Suite's default filters for CSS, images, and other static content to ensure all requests are captured and visible in the HTTP history.

This allows for comprehensive analysis of how the application handles file requests, including images and other static resources.

---

### Step 2 - Identifying the Vulnerable Endpoint

While browsing the application, examine the captured requests in Burp Suite's HTTP history. Look for GET requests that fetch images or files using a path parameter rather than serving files directly.

A request pattern that accepts a file path as a parameter indicates potential file inclusion functionality that may be vulnerable to path traversal attacks.

---

### Step 3 - Request Analysis in Repeater

Send the suspicious image retrieval request to Burp Suite's Repeater tool for manual testing. Analyze the request structure to understand how the file path parameter is constructed and processed by the server.

The parameter likely accepts a relative or absolute path that the server uses to locate and return the requested file.

---

### Step 4 - Path Traversal Testing

Test for path traversal by modifying the file path parameter using directory traversal sequences. Start with a basic payload:

```
../
```

If the application responds differently or returns an error, this confirms that the input is being processed as a file path. Progressively add more traversal sequences to move up the directory structure:

```
../../
../../../
../../../../
```

---

### Step 5 - Directory Enumeration and Flag Retrieval

Continue iterating through directory levels by adding additional `../` sequences until reaching the server's root or a directory containing sensitive files. Test common file locations such as:

```
../../../../flag.txt
../../../../etc/passwd
```

When the correct number of directory traversals is used, the server returns the contents of the target file instead of the expected image, revealing the flag.

---

### Impact
- Unauthorized access to arbitrary files on the server filesystem
- Exposure of sensitive configuration files and credentials
- Potential access to source code and application secrets
- Information disclosure enabling further attacks
- Complete bypass of intended file access restrictions
- Risk of accessing system files like /etc/passwd or application configuration

---

### Vulnerability Classification
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Path Traversal / Local File Inclusion (LFI)
- **Attack Surface:** Image retrieval endpoint with file path parameter
- **CWE:** CWE-22 - Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')

---

### Root Cause
The backend accepts user-supplied input for file path parameters without proper validation or sanitization. The application fails to restrict file access to the intended directory and does not filter or reject path traversal sequences like `../`. This allows attackers to escape the web root or designated file directory and access arbitrary files on the server filesystem.

The vulnerability exists because the developer trusted user input to remain within expected boundaries, rather than implementing a whitelist of allowed files or canonicalizing paths before access.

---

### Remediation
- Implement strict input validation to reject path traversal sequences (../, ..\, etc.)
- Use a whitelist of allowed files rather than accepting arbitrary paths
- Canonicalize file paths and verify they remain within the intended directory after resolution
- Implement proper access controls to restrict file access to designated directories
- Use indirect references (e.g., numeric IDs mapped to files) instead of direct file paths
- Configure the web server to run with minimal filesystem permissions
- Apply chroot or containerization to limit the accessible filesystem scope
- Log and monitor file access attempts for suspicious patterns
- Conduct regular security testing for path traversal vulnerabilities in file-handling functions

---
