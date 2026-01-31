---
layout: post
title:  "BugForge - Daily - Tanuki"
date:   2026-01-27 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor,broken-access-control]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki
><br/><b>Vulnerabilities Covered:</b>
<br/>
Insecure Direct Object Reference (IDOR)<br/>
Broken Access Control
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates an Insecure Direct Object Reference (IDOR) vulnerability in the profile update functionality. The application passes the username as a query parameter in the PUT request when updating profile details, allowing an attacker to modify other users' profiles by simply changing the username parameter to a target victim's username. By creating two test accounts and intercepting the profile update request, the vulnerability was confirmed when successfully modifying the second account's profile data using the first account's session. This highlights a critical access control failure where the server trusts client-supplied identifiers without verifying that the authenticated user owns the resource being modified.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
Begin by creating two separate user accounts to enable cross-account vulnerability testing. This setup allows us to verify whether actions performed by one account can affect another account's data. Document the usernames and credentials for both accounts as they will be needed during the exploitation phase.

---

### Step 2 - Profile Update Functionality Analysis
Navigate to the profile settings page and update the profile details for the first account.  Analyze the PUT request structure and observe how the application identifies which user's profile to update.

Notice that the username is passed as a query parameter in the request URL rather than being derived from the session or authentication token. This is a significant security concern as it allows client-side manipulation of the target resource identifier.

---

### Step 3 - Exploiting the IDOR Vulnerability
With the profile update request captured, modify the username query parameter from the first account's username to the second account's username. Keep the session token and authentication cookies from the first account unchanged. Submit the modified request to the server.

The server processes the request and updates the second account's profile using the data supplied by the first account, confirming the IDOR vulnerability. The flag is returned upon successful exploitation.

![Flag](/images/bug-forge/daily/Tanuki/idor-profile-update/flag.png)

---

### Impact
- Unauthorized modification of other users' profile information
- Account takeover potential if email or password fields can be modified
- Privacy violations through exposure or alteration of personal data
- Reputation damage if attacker modifies visible profile fields maliciously
- Potential for mass exploitation affecting all users on the platform
- Complete bypass of intended access control mechanisms

---

### Vulnerability Classification
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** Profile update API endpoint
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The backend fails to verify that the authenticated user owns the resource being modified. Instead of deriving the target user identity from the session token or authentication context, the application trusts the client-supplied username parameter in the request. This allows any authenticated user to specify an arbitrary username and modify that user's profile data. The server performs no authorization check to confirm that the requesting user has permission to modify the specified account.

---

### Remediation
- Derive the user identity from the server-side session rather than client-supplied parameters
- Implement proper authorization checks to verify resource ownership before processing modifications
- Use indirect references (such as UUIDs or session-bound identifiers) instead of predictable usernames
- Apply the principle of least privilege ensuring users can only access their own resources
- Add logging and monitoring for profile modification requests to detect anomalous patterns
- Implement rate limiting on sensitive endpoints to slow down enumeration attacks
- Conduct regular access control testing to identify IDOR vulnerabilities across all endpoints
- Use automated security scanning tools that specifically test for IDOR patterns

---
