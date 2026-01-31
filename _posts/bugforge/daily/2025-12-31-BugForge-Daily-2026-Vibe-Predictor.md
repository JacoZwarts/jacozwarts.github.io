---
layout: post
title:  "BugForge - Daily - 2026 Vibe Predictor"
date:   2025-12-31 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [ssti]
categories: [BugForge,daily,2026-vibe-predictor]
---

# Daily - 2026 Vibe Predictor
><br/><b>Vulnerabilities Covered:</b>
<br/>
SSTI (Server Side Template Injection)
<br/>
<br/>
<b>Summary:</b><br/>
This vulnerability is a **Server-Side Template Injection (SSTI)** issue where user-supplied input is evaluated by the EJS template engine. By injecting template expressions into the answer submission endpoint, an attacker can execute server-side logic and bypass filtering controls. Even when direct file system access is blocked, access to runtime objects such as environment variables enables sensitive data disclosure and creates the potential for full server compromise.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Identifying Template Injection

The answer submission endpoint was tested for server-side template injection by submitting a simple mathematical expression commonly used to detect SSTI behavior.

```json
{"questionId":"name","answer":"<%= 7*7 %>"}
```

The response evaluated the expression and returned the calculated result, confirming that user-supplied input was being executed server-side rather than treated as static data.

---

### Step 2 - Confirming the Template Engine

Additional expressions were submitted to determine the underlying template engine in use.  
The application evaluated these expressions according to EJS syntax, confirming that the backend was rendering user input using the EJS template engine.

---

### Step 3 - Attempting File System Access

With SSTI confirmed, attempts were made to access the server file system to retrieve sensitive files.  
These requests were blocked by application filters or a WAF, indicating that direct file system access was restricted.

```json
{"questionId":"name","answer":"<%= require('fs').readFileSync('/flag.txt', 'utf8') %>"}
```

---

### Step 4 - Bypassing Filters via Environment Variables

As file system access was blocked, the next step was to test whether runtime environment variables were accessible within the template context.  
The response confirmed that environment variables were available, although not directly readable in their raw form.

```json
{"questionId":"name","answer":"<%= process.env %>"}
```

---

### Step 5 - Extracting Environment Variable Contents

The environment variables were then coerced into a readable format by stringifying the runtime object.  
This caused the application to return the full set of environment variables in an encoded format, including sensitive values.

```json
{"questionId":"name","answer":"<%= JSON.stringify(process.env) %>"}
```

---

### Step 6 - Flag Extraction and Decoding

Within the encoded response, the flag value was identified and decoded from its encoded representation.  
Decoding the value revealed the flag and completed the challenge.

---

### Impact
- Execution of arbitrary template expressions on the server  
- Exposure of sensitive environment variables  
- Leakage of secrets such as tokens, credentials, or flags  
- Bypass of filtering and WAF controls  
- Demonstrates a high-impact injection vulnerability  

---

### Vulnerability Classification
- **OWASP Top 10:** Injection  
- **Vulnerability Type:** Server-Side Template Injection (SSTI)  
- **Attack Surface:** Answer submission API  
- **CWE:** CWE-1336 - Improper Neutralization of Special Elements Used in a Template Engine  

---

### Root Cause

The application evaluates user-controlled input directly within an EJS template without proper sanitization, sandboxing, or variable allowlisting.  
This allows attackers to execute arbitrary template logic and access sensitive runtime objects such as environment variables, even when certain operations are filtered.

---

### Remediation
- Do not evaluate untrusted user input inside server-side templates  
- Use strict variable binding instead of raw template expression evaluation  
- Restrict access to global runtime objects within templates  
- Implement allowlists for permitted template variables  
- Apply robust input validation and output encoding  
- Include SSTI testing as part of the secure development lifecycle  

---