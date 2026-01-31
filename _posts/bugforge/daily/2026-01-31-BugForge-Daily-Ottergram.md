---
layout: post
title:  "BugForge - Daily - Ottergram"
date:   2026-01-31 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor,broken-access-control]
categories: [BugForge,daily,ottergram]
---

# Daily - Ottergram
><br/><b>Vulnerabilities Covered:</b>
<br/>
Insecure Direct Object Reference (IDOR)
<br/>
Broken Access Control
<br/>
<br/>
<b>Summary:</b>
<br/>
The Ottergram application contains an **`Insecure Direct Object Reference (IDOR)`** vulnerability in its profile update functionality. While the application properly authenticates users via JWT tokens, the profile update endpoint fails to verify that the user ID in the request body belongs to the authenticated user. By retrieving the admin user's ID through the publicly accessible `/api/profile/{username}` endpoint and substituting it in a profile update request, an attacker can modify any user's profile information including the administrator's account. This demonstrates a critical broken access control flaw where authorization checks are missing on sensitive operations.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
Analyze the application functionality to identify potential attack vectors. The application allows users to create posts with images and descriptions, leave comments, and like other posts. Testing JWT token manipulation reveals no exploitable weaknesses in the authentication mechanism. The remaining functionality to investigate is the profile update feature.

---

### Step 2 - Profile Endpoint Enumeration
Navigate to the profile page and observe the network requests. The application calls `/api/profile/{username}` to retrieve user data.

![Profile - UI](/images/bug-forge/daily/ottergram/idor-profile-update/profile-ui.png)

![Get Profile request](/images/bug-forge/daily/ottergram/idor-profile-update/get-profile-request.png)

Request the admin user's profile by changing the username parameter to `admin`. This reveals the admin's user ID, which will be needed for the IDOR attack.

![Admin Profile](/images/bug-forge/daily/ottergram/idor-profile-update/get-profile-admin-request.png)

---

### Step 3 - Profile Update Request Analysis
Update your own profile and intercept the request using a proxy tool. Notice that the request body contains the user's ID as a parameter, which the server uses to determine which profile to update.

![Profile Update](/images/bug-forge/daily/ottergram/idor-profile-update/profile-update.png)

---

### Step 4 - IDOR Exploitation
Modify the profile update request by replacing your user ID with the admin's user ID obtained in Step 2. Change the `fullname` and `bio` fields to arbitrary values and submit the request.

![Update Admin Profile](/images/bug-forge/daily/ottergram/idor-profile-update/update-admin-profile-request.png)

---

### Step 5 - Verify Exploitation
Retrieve the admin's profile again to confirm the changes were applied. The modified profile data confirms successful exploitation of the IDOR vulnerability, and the flag is revealed in the response.

![Flag](/images/bug-forge/daily/ottergram/idor-profile-update/flag.png)

---

### Impact
- Unauthorized modification of any user's profile data including administrators
- Potential for account takeover through profile field manipulation (email, linked accounts)
- Reputational damage through defacement of user profiles
- Privacy violation by exposing user data modification capabilities
- Horizontal privilege escalation across all application users
- Vertical privilege escalation if profile fields control access levels

---

### Vulnerability Classification
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** Profile update API endpoint
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The profile update endpoint accepts a user ID directly from the request body without validating that it matches the authenticated user's session. The server trusts the client-supplied user ID to determine which profile to modify, rather than extracting the user identity from the authenticated session token. This allows any authenticated user to update any other user's profile by simply changing the ID parameter in their request.

---

### Remediation
- Extract user identity from the authenticated session token rather than accepting it as a request parameter
- Implement server-side authorization checks to verify the authenticated user owns the resource being modified
- Use indirect object references (such as mapping to internal IDs server-side) instead of exposing direct database identifiers
- Apply the principle of least privilege by restricting profile modifications to the profile owner only
- Log and monitor for unauthorized access attempts to detect exploitation
- Implement rate limiting on sensitive endpoints to slow down enumeration attacks
- Consider using UUIDs instead of sequential IDs to make enumeration more difficult (defense in depth, not a fix)

---
