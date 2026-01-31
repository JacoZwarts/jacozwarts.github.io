---
layout: post
title:  "BugForge - Daily - Ottergram"
date:   2026-01-03 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli]
categories: [BugForge,daily,ottergram]
---

# Daily - Ottergram
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQL Injection (UNION-based)
<br/>
<br/>
<b>Summary:</b>
<br/>
A **`SQL Injection`** vulnerability was identified in the user profile retrieval functionality where user-supplied input is concatenated directly into SQL queries without proper parameterization or input validation. By injecting a single quote into the profile endpoint, error-based confirmation of the vulnerability was achieved. Using the ORDER BY technique to enumerate the number of columns returned by the query, a UNION-based SQL injection attack was constructed to extract sensitive data from the underlying database. This allowed retrieval of the username and password fields from the users table, ultimately exposing the administrator credentials and the challenge flag.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Reconnaissance

Analyze the application's functionality and identify features that interact with the backend database. The profile retrieval functionality accepts user input to fetch profile data, making it a strong candidate for injection testing.

---

### Step 2 - Initial SQL Injection Test

Test the profile endpoint for SQL injection by appending a single quote (`'`) to the input parameter. Observe the application's response for database errors or abnormal behavior that indicates the input is being interpreted as part of a SQL query.

A database error or unexpected response confirms that user input is being concatenated into the SQL query without proper sanitization.

---

### Step 3 - Column Enumeration

Use the `ORDER BY` technique to determine the number of columns returned by the underlying SQL query. Incrementally increase the column number until an error occurs:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

When the ORDER BY clause references a column number that exceeds the actual count, the application will return an error, revealing the exact number of columns in the result set.

---

### Step 4 - UNION-Based Data Extraction

With the column count confirmed, construct a UNION-based SQL injection payload to extract sensitive data from the database. Target the `users` table to retrieve the `username` and `password` fields:

```
' UNION SELECT username, password FROM users--
```

Adjust the number of NULL placeholders as needed to match the column count identified in the previous step.

---

### Step 5 - Flag Discovery

Review the extracted data returned in the application response. The administrator account credentials and the challenge flag are revealed within the query results.

![Flag](/images/bug-forge/daily/ottergram/sqli-profile/flag-response.png)

---

### Impact
- Unauthorized access to sensitive database contents including user credentials
- Complete bypass of authentication mechanisms through credential extraction
- Potential exposure of all user data stored in the database
- Risk of privilege escalation by obtaining administrator credentials
- Demonstrates a critical failure in input validation and query construction

---

### Vulnerability Classification
- **OWASP Top 10:** Injection
- **Vulnerability Type:** SQL Injection (UNION-based)
- **Attack Surface:** Profile retrieval endpoint with unsanitized user input
- **CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command

---

### Root Cause
The application constructs SQL queries by directly concatenating user-supplied input without using parameterized queries or prepared statements. No input validation or sanitization is performed on the profile identifier before it is included in the database query, allowing attackers to manipulate the query structure and inject arbitrary SQL commands.

---

### Remediation
- Use parameterized queries or prepared statements for all database interactions
- Implement strict input validation to reject unexpected characters and patterns
- Apply the principle of least privilege to database accounts used by the application
- Deploy a Web Application Firewall (WAF) to detect and block common injection patterns
- Conduct regular security testing to identify injection vulnerabilities before deployment
- Avoid displaying detailed database error messages to end users

---
