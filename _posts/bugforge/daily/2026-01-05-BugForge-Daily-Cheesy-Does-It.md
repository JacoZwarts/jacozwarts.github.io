---
layout: post
title:  "BugForge - Daily - Cheesy Does It"
date:   2026-01-05 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQL Injection (Authentication Bypass)
<br/>
<br/>
<b>Summary:</b>
<br/>
The login functionality is vulnerable to SQL Injection, allowing attackers to bypass authentication and gain unauthorized access to admin accounts. By injecting a payload such as `' or 1=1--` into the username field, the attacker manipulates the underlying SQL query to always evaluate as true, effectively bypassing password validation. This classic authentication bypass vulnerability grants immediate access to privileged accounts and exposes the challenge flag.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Exploration
Launch the lab and navigate to the application. The initial page redirects to a login form. Since no credentials are available, register a new account using the "Register here" option to explore the application functionality.

---

### Step 2 - Understanding the Application
After registration, the application automatically logs you in and reveals a pizza ordering website with several features including Build Pizza, Orders, Cart, Profile, and Payment functionality. This provides context for the application but the vulnerability lies in the authentication mechanism.

---

### Step 3 - Analyze the Login Request
Open Burp Suite or Caido and intercept the login request. The `POST /api/login` endpoint accepts a JSON payload containing username and password fields. A normal login attempt with invalid credentials returns an "Invalid credentials" error message.

---

### Step 4 - Test for SQL Injection
Test the login form for SQL injection vulnerability by injecting a basic payload into the username field. Use a payload such as:

```
' or 1=1--
```

or

```
admin'--
```

These payloads work by closing the string literal with a single quote, adding an always-true condition (`or 1=1`) or commenting out the password check entirely (`--`).

---

### Step 5 - Authentication Bypass
Send the modified login request with the SQL injection payload. The server accepts the manipulated query and authenticates the request, bypassing the password validation entirely. The query now evaluates as:

```sql
SELECT * FROM users WHERE username='admin' OR 1=1--' AND password='anything'
```

Since `1=1` always evaluates to true, the database returns the first user (typically admin), granting unauthorized access.

---

### Step 6 - Flag Retrieval
Upon successful authentication bypass, the application grants admin access and immediately reveals the challenge flag, confirming the SQL injection vulnerability in the authentication mechanism.

---

### Impact
- Complete authentication bypass without valid credentials
- Unauthorized access to administrative accounts
- Full compromise of user account security
- Potential for data exfiltration from the underlying database
- Foundation for further SQL injection attacks (data extraction, modification, deletion)

---

### Vulnerability Classification
- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** SQL Injection (Authentication Bypass)
- **Attack Surface:** Login API endpoint (`POST /api/login`)
- **CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command

---

### Root Cause
The backend constructs SQL queries by directly concatenating user-supplied input without proper sanitization or parameterization. The login query likely follows a pattern such as `SELECT * FROM users WHERE username='[input]' AND password='[input]'`, allowing attackers to inject malicious SQL syntax that alters the query logic and bypasses authentication controls.

---

### Remediation
- Use parameterized queries (prepared statements) for all database operations
- Implement input validation to reject unexpected characters in authentication fields
- Apply the principle of least privilege for database accounts used by the application
- Deploy a Web Application Firewall (WAF) to detect and block common injection patterns
- Implement account lockout mechanisms after failed login attempts
- Add comprehensive logging and monitoring for authentication events
- Conduct regular security testing including SQL injection testing as part of the SDLC

---
