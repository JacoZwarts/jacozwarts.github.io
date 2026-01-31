---
layout: post
title:  "BugForge - Daily - Copy Pasta"
date:   2026-01-07 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor,broken-access-control]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Copy Pasta
><br/><b>Vulnerabilities Covered:</b>
<br/>
IDOR (Insecure Direct Object Reference)
<br/>
<br/>
<b>Summary:</b>
<br/>
The CopyPasta application allows users to create and share code snippets with options to make them public or private. The snippet retrieval endpoint (`/api/snippets/:id`) uses sequential integer identifiers without proper authorization checks, allowing any authenticated user to enumerate IDs and access private snippets belonging to other users. By iterating through snippet IDs using Burp Intruder, private snippets containing sensitive data including the challenge flag can be retrieved, demonstrating a classic IDOR vulnerability on the read operation.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Initial Reconnaissance
Before signing up, examine the login page source code for any developer comments or hidden information. While this challenge did not reveal anything in the source, always check for horizontal scrolling as CTFs sometimes hide clues off-screen.

Attempt basic SQL injection on the login form using payloads such as `admin' OR 1=1;-- -`. The application returns a generic "Invalid credentials" message, indicating that username enumeration via error messages is not possible.

---

### Step 2 - Account Registration and Traffic Analysis
Register a new user account while capturing traffic in Burp Suite. Key observations from the registration request/response:

- The registration endpoint uses `/api/register`, indicating an API-driven architecture
- JWT (JSON Web Token) authentication is in use
- The response includes a `role` field set to `user`, which was not present in the request (potential mass assignment vector for future testing)

---

### Step 3 - Application Mapping
After logging in, explore the application functionality:

- **My Snippets / Dashboard**: Create and manage personal code snippets
- **Public**: View publicly shared snippets from all users with search and language filter options
- **Create Snippet**: Form with title, language, code content, and visibility toggle (public/private)

Create a test snippet and observe the response. Note the `id` field (sequential integer) and `share_code` field (long token string) returned in the response.

---

### Step 4 - Identifying the IDOR Vector
When viewing a public snippet, observe that the URL follows the pattern `/snippet/:id` and the API endpoint is `/api/snippets/:id`.

Key observations:
- The public dashboard shows 6 snippets
- When creating your own snippet, the returned `id` value suggests there are more snippets than publicly visible (e.g., id of 8 indicates snippets 1-7 already exist)
- This discrepancy suggests some snippets are private and potentially accessible via direct ID reference

---

### Step 5 - Exploiting the IDOR Vulnerability
Send the GET request for `/api/snippets/1` to Burp Intruder. Configure the attack:

- Set the snippet ID as the payload position
- Use a numeric payload list (e.g., 1-100)
- Remove the `If-None-Match` header if present to avoid 304 responses

Launch the attack and analyse the responses. Filter results by response length or status code to identify unique snippets.

---

### Step 6 - Flag Retrieval
Review the Intruder results to identify snippets that are not visible in the public dashboard. One of the private snippets contains the flag in the response body.

The flag follows the format `bug{FLAG}` and can be found by:
- Comparing Intruder results against the public snippet list
- Searching response bodies for the flag format
- Identifying snippets where `public: 0` indicates private status

---

### Impact
- Unauthorized access to private snippets belonging to other users
- Exposure of potentially sensitive code or data stored in private snippets
- Complete bypass of intended access control boundaries

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** Snippet retrieval endpoint (`/api/snippets/:id`)
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The backend does not verify snippet ownership or visibility permissions when processing GET requests for individual snippets. The application relies solely on user-supplied identifiers without performing authorization checks to ensure the requesting user has permission to view the resource.

---

### Remediation
- Implement server-side authorization checks to verify that the authenticated user owns the snippet or the snippet is marked as public before returning its contents
- Consider using non-sequential identifiers such as UUIDs to make enumeration more difficult (though this is defense-in-depth, not a substitute for proper authorization)
- Apply consistent access control checks across all snippet-related endpoints (view, update, delete)
- Log and monitor for unusual patterns of sequential ID access that may indicate enumeration attempts

---
