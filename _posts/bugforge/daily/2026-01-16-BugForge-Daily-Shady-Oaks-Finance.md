---
layout: post
title:  "BugForge - Daily - Shady Oaks Finance"
date:   2026-01-16 20:40
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control]
categories: [BugForge,daily,shady-oaks-finance]
---

# Daily - Shady Oaks Finance
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Access Control
<br/>
<br/>
<b>Summary:</b>
<br/>
**`Broken access control`** was identified where administrative endpoints were exposed without proper server-side authorization checks. By enumerating application endpoints and directly accessing admin/* routes as a standard user, it was possible to reach privileged functionality and retrieve sensitive data without an admin role. This issue highlights that even basic access control assumptions can fail and reinforces the **`importance of always testing fundamental authorization controls`** before moving on to more complex attack paths, as simple checks often lead to critical findings.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution
### Step 1 - Account Creation

Inspect the user registration request and response to identify any roles or privilege-related fields being assigned during account creation.

![User registration request](/images/bug-forge/daily/shady-oaks-financial/broken-access-control/user-registration-request.png)

---

### Step 2 - Endpoint Analysis

Use **JS Recon Buddy** to enumerate and analyze available application endpoints. During this process, identify endpoints that appear to be restricted to administrative users.

![Discovered admin endpoints](/images/bug-forge/daily/shady-oaks-financial/broken-access-control/js-recon-buddy-analyse-endpoints.png)

---

### Step 3 - Unauthorized Access to Admin Endpoint

Attempt to access the `admin/users` endpoint directly while authenticated as a standard user, without any administrative role assigned.

![Accessing admin users endpoint](/images/bug-forge/daily/shady-oaks-financial/broken-access-control/endpoint-admin-users.png)

The request succeeds, confirming that the endpoint is accessible without proper authorization checks.

---

### Step 4 - Accessing the Admin Flag

After confirming the presence of broken access control, directly request the `admin/flag` endpoint.

![Admin flag endpoint](/images/bug-forge/daily/shady-oaks-financial/broken-access-control/flag.png)

The flag is returned, demonstrating a critical authorization failure due to missing role-based access enforcement.

---

### Impact
- Unauthorized access to administrative functionality
- Exposure of sensitive application data and privileged endpoints
- Potential for full privilege escalation by any authenticated user
- Indicates a complete failure of role-based access control enforcement

---

### Vulnerability Classification
- OWASP Top 10: Broken Access Control
- Vulnerability Type: Missing Function-Level Authorization
- CWE: CWE-285 - Improper Authorization

---

### Root Cause
The backend does not enforce server-side authorization checks on administrative endpoints and relies on assumed user roles rather than validating permissions for each request.

---

### Remediation
- Enforce strict server-side authorization checks on all admin endpoints
- Validate user roles and permissions for every privileged request
- Deny access by default and explicitly allow only authorized roles
- Avoid relying on client-side logic or endpoint obscurity for access control
- Regularly test basic access control scenarios, including direct endpoint access as non-privileged users

---