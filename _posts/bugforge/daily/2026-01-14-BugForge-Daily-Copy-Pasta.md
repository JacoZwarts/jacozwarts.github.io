---
layout: post
title:  "BugForge - Daily - CopyPasta"
date:   2026-01-14 19:48
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,copy-pasta]
---

# Daily - CopyPasta
><br/><b>Vulnerabilities Covered:</b>
<br/>
IDOR (Insecure Direct Object Reference)
<br/>
<br/>
<b>Summary:</b>
<br/>
After registering a standard user, the application was mapped to understand how snippets are created and managed, with a focus on how snippet IDs are handled across API endpoints. Attempts to modify other users’ snippets via ID manipulation in update requests were correctly restricted, and basic injection testing yielded no results. However, the **`delete`** functionality passed the snippet ID as a query parameter without proper ownership checks, allowing enumeration of IDs and unauthorized deletion of other users’ snippets, ultimately leading to flag retrieval via an IDOR in the delete endpoint.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Registration
Register a new user account using the application’s standard registration flow.

![Account Registration - UI](/images/bug-forge/daily/copy-pasta/idor-delete/account-registration.png)

Analyse the registration request, to see if any roles are passed when creating a new user.
![Account Registration - Request](/images/bug-forge/daily/copy-pasta/idor-delete/account-registration-request-response.png)
---

### Step 2 - Application Reconnaissance
Navigate through the application to understand how snippets are created, viewed, updated, and deleted.

Focus areas:
- Snippet ownership
- API endpoints handling snippet actions
- How snippet identifiers are passed to the backend

![App endpoints - Functionality](/images/bug-forge/daily/copy-pasta/idor-delete/app-endpoint-functionality.png)

---

### Step 3 - Unauthorised Snippet Update Attempt
Attempt to update another user’s snippet by manipulating the snippet identifier.

Test performed:
- Intercept the PUT request used to update a snippet
- Modify the `:id` value to reference a snippet belonging to another user

Result:
- The update attempt failed or was properly restricted

![Unauthorised snippet update attempt](/images/bug-forge/daily/copy-pasta/idor-delete/edit-snippet-unauthorised-response.png)

---

### Step 4 - Input Injection Testing
Attempt SQL injection in the snippet comment field.

Result:
- No exploitable SQL injection identified
- Input appears to be handled or sanitized correctly

![Comment - XSS attempt](/images/bug-forge/daily/copy-pasta/idor-delete/snippet-comment-xss-attempt.png)

---

### Step 5 - Unauthorized Snippet Deletion
Analyze the delete functionality to identify authorization weaknesses.

Observation:
- The snippet `id` is passed as a query parameter in the delete request

Exploitation:
- Iterate through different snippet IDs
- Successfully delete snippets belonging to other users
- 
![Attempt to delete snippet (IDOR)](/images/bug-forge/daily/copy-pasta/idor-delete/testing-delete-idor.png)

---

### Step 6 - Flag Retrieval
During ID enumeration, delete a snippet owned by another user containing the flag.

Outcome:
- Flag successfully retrieved through unauthorized snippet deletion

![Flag](/images/bug-forge/daily/copy-pasta/idor-delete/flag.png)
---

### Impact
- Unauthorized deletion of other users’ private snippets
- Loss of data integrity
- Indicates broken authorization controls

---

### Vulnerability Classification
- OWASP Top 10: Broken Access Control
- Vulnerability Type: Insecure Direct Object Reference (IDOR)
- CWE: CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The backend does not validate snippet ownership before processing delete requests and relies solely on user-supplied identifiers.

---

### Remediation
- Enforce server-side authorization checks on all snippet actions
- Validate resource ownership before update or delete operations
- Avoid exposing direct object identifiers where possible
- Apply consistent access control checks across all endpoints

---