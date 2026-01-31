---
layout: post
title:  "BugForge - Daily - Sokudo"
date:   2026-01-08 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-authentication,information-disclosure,session-hijacking]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Authentication
Information Disclosure
Session Hijacking
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates a `broken authentication` vulnerability caused by predictable session tokens combined with `information disclosure`. The application generates authentication tokens using the user's login timestamp in `YYYYMMDDHHMMSS` format, which is a weak and predictable value. The `/api/stats/leaderboard` endpoint exacerbates this vulnerability by exposing the `last_login` timestamps for all users, including administrators. By extracting the admin's login timestamp from the leaderboard response and formatting it as a token, an attacker can replace their own token in localStorage to hijack the administrator's session and gain access to privileged functionality.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Registration and Token Analysis
A new account was registered while monitoring the HTTP traffic using a proxy tool (Burp Suite or Caido). Upon successful registration, the server response revealed an authentication token that immediately stood out as potentially vulnerable.

The token value (e.g., `20260107090905`) appeared to be a UTC timestamp in the format `YYYYMMDDHHMMSS`. Cross-referencing the token with the current time confirmed this hypothesis - the token represented the exact moment of login.

**Key Observation:** Authentication tokens should be cryptographically random and unpredictable. A timestamp-based token is fundamentally insecure because:
- It can be predicted if the attacker knows or can estimate when a user logged in
- It provides no entropy beyond the second-level precision of the login time

---

### Step 2 - Application Exploration
After logging in, the application presents a speed typing test interface. Users can complete typing challenges and view their performance statistics. The application features:

- A typing test module for measuring words per minute (WPM)
- A personal statistics dashboard
- A leaderboard showing top performers

A typing test was completed to generate activity data and explore all available functionality.

---

### Step 3 - Information Disclosure via Leaderboard Endpoint
While reviewing the leaderboard functionality, the `/api/stats/leaderboard` endpoint was identified as a critical information disclosure vulnerability. The API response included a `last_login` field for each user on the leaderboard:

```json
{
  "leaderboard": [
    {
      "username": "admin",
      "last_login": "2026-01-07T15:04:59.000Z",
      "wpm": 95
    },
    {
      "username": "testuser",
      "last_login": "2026-01-07T09:09:05.000Z",
      "wpm": 45
    }
  ]
}
```

This response leaks sensitive authentication-related information. Combined with the knowledge that tokens are timestamp-based, this disclosure allows an attacker to reconstruct any user's session token.

---

### Step 4 - Token Reconstruction and Session Hijacking
Using the admin's `last_login` value from the leaderboard response, the authentication token was reconstructed:

1. **Extract the timestamp:** `2026-01-07T15:04:59.000Z`
2. **Convert to token format:** Remove all separators and formatting to produce `20260107150459`

The token was tested against the `/api/verify-token` endpoint to confirm validity before proceeding with the session hijack.

---

### Step 5 - Exploiting the Vulnerability
To complete the attack:

1. Open the browser's Developer Tools (F12)
2. Navigate to the Application/Storage tab
3. Select Local Storage for the application domain
4. Locate the authentication token field
5. Replace the current token value with the admin's reconstructed token (`20260107150459`)
6. Refresh the page

Upon refresh, the application recognized the session as belonging to the admin user. A new "Admin Panel" option appeared in the navigation, and accessing it revealed the flag.

---

### Impact
- Complete session hijacking of any user whose login time is exposed
- Unauthorized access to administrative functionality and sensitive data
- Potential for mass account takeover if login times are systematically harvested
- Bypasses all authentication controls without requiring credentials
- Demonstrates how weak token generation combined with information disclosure creates critical vulnerabilities

---

### Vulnerability Classification
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** Broken Authentication, Predictable Session Tokens
- **Secondary Issue:** Information Disclosure (Sensitive Data Exposure)
- **CWE:** CWE-330 - Use of Insufficiently Random Values
- **CWE:** CWE-200 - Exposure of Sensitive Information to an Unauthorized Actor

---

### Root Cause
The application made two critical security mistakes:

1. **Weak Token Generation:** Authentication tokens were derived from login timestamps rather than cryptographically secure random values. This makes tokens predictable if an attacker can determine or guess when a user logged in.

2. **Information Disclosure:** The leaderboard API unnecessarily exposed `last_login` timestamps for all users, providing attackers with the exact information needed to reconstruct valid session tokens.

---

### Remediation
- Generate session tokens using cryptographically secure random number generators (CSPRNG)
- Ensure tokens have sufficient entropy (minimum 128 bits recommended)
- Remove sensitive fields like `last_login` from public API responses unless explicitly required
- Implement proper session management with server-side validation
- Consider using established session management frameworks or libraries (e.g., JWT with proper signing)
- Add rate limiting to prevent token enumeration attempts
- Implement session binding to additional factors (IP address, user agent) where appropriate
- Monitor for suspicious authentication patterns such as token reuse from different locations
- Conduct regular security reviews of authentication mechanisms

---
