---
layout: post
title:  "BugForge - Daily - Sokudo"
date:   2026-01-29 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-authentication]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Authentication
<br/>
<br/>
<b>Summary:</b>
<br/>
The Sokudo application uses predictable ISO 8601 timestamps as authentication tokens, creating a critical broken authentication vulnerability. By analyzing the application's responses, it was discovered that the Bearer token is simply the user's registration timestamp in `YYYYMMDDHHmmss` format. Combined with an information disclosure vulnerability in the leaderboard endpoint that exposes each user's `last_login` timestamp, an attacker can derive any user's authentication token. By converting the admin user's last login time to the ISO 8601 format and using it as a Bearer token, complete administrative access was achieved, demonstrating how predictable tokens and excessive data exposure can lead to full account takeover.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Registration
Create a new account and notice the first endpoint trigger `/api/stats` observe the Authentication Bearer Token `20260129092710` this looks like an ISO 8601 timestamp.
![ISO 8601 Timestamp as Auth Token](/images/bug-forge/daily/Sokudo/broken-authentication/authentication-token.png)

---

### Step 2 - Leaderboard analysis
Navigate to the leader board functionality and observe the backend call. Notice that the leaderboard endpoint overshares data and shows the user's last login date. Notice the property for the admin user `last_login` we'll convert the DateTime property to a ISO 8601 timestamp `20260129092645`.

![Admin last login log](/images/bug-forge/daily/Sokudo/broken-authentication/admin-last-login.png)

Update the browser storage to have the admin's token, refresh the page and notice we've unlocked the Admin feature on the menu.

Notice the `/api/admin/users` request and the response. notice the flag in the response.

![Flag](/images/bug-forge/daily/Sokudo/broken-authentication/flag.png)
---

### Impact
- Complete account takeover of any user including administrators
- Unauthorized access to administrative functions and sensitive data
- Ability to impersonate any user by calculating their authentication token
- Full bypass of authentication mechanisms without requiring credentials
- Potential for mass exploitation as all user tokens follow the same predictable pattern
- Exposure of user activity patterns through leaked last login timestamps

---

### Vulnerability Classification
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** Broken Authentication / Predictable Tokens
- **Attack Surface:** Authentication token generation, Leaderboard API endpoint (`/api/stats`)
- **CWE:** CWE-330 - Use of Insufficiently Random Values, CWE-200 - Exposure of Sensitive Information

---

### Root Cause
The application generates authentication tokens using predictable timestamps rather than cryptographically secure random values. The Bearer token is simply the user's registration or login timestamp converted to ISO 8601 format (`YYYYMMDDHHmmss`), making it trivially guessable. Additionally, the leaderboard endpoint (`/api/stats`) returns excessive user information including the `last_login` timestamp, providing attackers with the exact value needed to compute valid authentication tokens for any user. The combination of predictable token generation and information disclosure creates a complete authentication bypass.

---

### Remediation
- Generate authentication tokens using cryptographically secure random number generators (CSPRNG)
- Use industry-standard token formats such as JWT with proper signing and expiration
- Remove unnecessary user data from public API responses (principle of data minimization)
- Never expose timestamps or other information that could be used to derive authentication credentials
- Implement token expiration and rotation policies to limit the window of exploitation
- Add rate limiting and anomaly detection for authentication endpoints
- Use secure session management libraries rather than custom token implementations
- Conduct regular security audits focusing on authentication and authorization mechanisms

---
