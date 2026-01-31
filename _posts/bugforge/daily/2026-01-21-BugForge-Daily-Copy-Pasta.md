---
layout: post
title:  "BugForge - Daily - Copy Pasta"
date:   2026-01-21 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,idor]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Access Control
IDOR (Insecure Direct Object Reference)
<br/>
<br/>
<b>Summary:</b>
<br/>
This issue is a classic example of `broken access control` caused by trusting user-supplied object identifiers. A password reset endpoint accepts a userId parameter without verifying that the authenticated user is authorized to act on that account, allowing an attacker to reset passwords for other users, including administrators. Public API responses further leak usernames, making it trivial to identify high-value targets. By chaining identifier manipulation with information disclosure, an attacker can achieve full account takeover and access sensitive application data, demonstrating an `insecure direct object reference (IDOR)` on critical account management functionality.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation and Baseline Review
A new user account was registered while intercepting both the request and response. The objective of this step was to establish a baseline understanding of the account creation flow and to identify any role, privilege, or identifier fields that could later influence authorization or access control decisions.

![Registration Request](/images/bug-forge/daily/copy-pasta/broken-access-control/registration-request.png)

---

### Step 2 - Snippet Functionality Testing
Testing initially focused on all snippet-related functionality. This included attempts to update and delete snippets belonging to other users, as well as adding comments containing common injection payloads such as cross-site scripting and server-side template injection. These tests did not result in unauthorized access or unexpected behavior.

---

### Step 3 - Profile Functionality Testing
After no issues were identified within the snippet functionality, testing shifted to user profile endpoints. Multiple actions were attempted, including modifying another user’s profile, deleting profiles, and resetting user passwords.

During this phase, the password reset endpoint was observed to accept a `userId` parameter. By modifying this parameter to reference another user’s identifier, it was possible to reset the password for a different account. Using this approach, the password for user ID 1 was successfully changed.

![Update user password](/images/bug-forge/daily/copy-pasta/broken-access-control/update-user-password.png)

Further investigation revealed that usernames were exposed in responses from the `/api/snippets/public` endpoint. This allowed the username associated with user ID 1 to be identified as `admin`.

![Username leak](/images/bug-forge/daily/copy-pasta/broken-access-control/snippet-response-username.png)

---

### Step 4 - Logging in as Administrator
With the administrator password successfully reset, it was possible to authenticate as the admin user. Upon logging in, the protected endpoint returned sensitive data, including the challenge flag, within the `/api/snippets` response.

![Flag](/images/bug-forge/daily/copy-pasta/broken-access-control/flag.png)

---

### Impact
- Unauthorized password reset for arbitrary user accounts  
- Full account takeover of privileged users, including administrators  
- Exposure of sensitive application data  
- Complete compromise of access control boundaries  

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control  
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)  
- **Attack Surface:** User profile and password reset endpoints  
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key  

---

### Root Cause
The backend trusted client-supplied identifiers such as `userId` without verifying that the authenticated user was authorized to perform actions on the referenced account. Authorization checks were either missing or insufficiently enforced on sensitive endpoints, allowing direct manipulation of object references.

---

### Remediation
- Enforce strict server-side authorization checks on all user-related actions  
- Ensure that sensitive operations such as password resets are strictly bound to the authenticated user’s session and identity, and require additional verification such as a one-time password (OTP) or confirmation of the user’s existing password before allowing the action to proceed.
- Avoid exposing internal identifiers or usernames in public API responses  
- Implement centralized access control logic and perform regular authorization testing  
- Add logging and monitoring for suspicious account management activity  

---