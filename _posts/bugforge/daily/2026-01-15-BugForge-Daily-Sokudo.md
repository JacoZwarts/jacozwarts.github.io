---
layout: post
title:  "BugForge - Daily - Sokudo"
date:   2026-01-15 20:40
image:  /images/bug-forge/bugforge-logo.png
tags:   [api-versioning,broken-authentication,idor,jwt-manipulation]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo
><br/><b>Vulnerabilities Covered:</b>
<br/>
API Versioning Vulnerability
Broken Authentication
IDOR (Insecure Direct Object Reference)
JWT Token Manipulation
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates how legacy API endpoints can introduce critical security vulnerabilities when not properly deprecated or secured. The application exposes both `/v1` and `/v2` API versions, where the newer `/v2` endpoints enforce proper JWT validation while the legacy `/v1` endpoints fail to verify token signatures and role claims. By discovering the `/v1/admin/flag` endpoint through enumeration and manipulating JWT claims (changing the role to "admin" and user ID to "1"), an attacker can bypass authentication and authorization controls to access administrative functionality that would otherwise be protected.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Initial Reconnaissance and API Version Discovery
After registering an account and logging in, all HTTP traffic was monitored using a proxy tool (Burp Suite). Analysis of the requests revealed that all API endpoints used the `/v2` prefix (e.g., `/v2/login`, `/v2/stats`, `/v2/register`).

This immediately prompted investigation into whether older API versions existed. Testing the same endpoints with `/v1` instead of `/v2` confirmed that legacy endpoints were still accessible.

**Key Observation:** Applications that expose multiple API versions without proper deprecation or security parity create opportunities for attackers to bypass newer security controls by targeting older implementations.

---

### Step 2 - Endpoint Enumeration
To identify all available endpoints across both API versions, Jason Haddix's endpoint discovery script was executed against the application. This enumeration revealed several endpoints, with particular interest in routes tagged with "admin" functionality.

The enumeration uncovered the following key endpoints:
- `/v2/admin/flag` - Returns "INVALID TOKEN" when accessed with a regular user token
- `/v1/admin/flag` - Returns "403 Forbidden" when accessed with a regular user token

**Key Observation:** The different error responses between versions indicated a significant behavioral difference. The `/v2` endpoint properly validated the JWT token and rejected it as invalid, while the `/v1` endpoint appeared to accept the token but denied access based on authorization (403 Forbidden).

---

### Step 3 - Analyzing the Authentication Difference
The distinction in error responses revealed a critical vulnerability:

| Endpoint         | Response      | Implication                                          |
| ---------------- | ------------- | ---------------------------------------------------- |
| `/v2/admin/flag` | INVALID TOKEN | Full JWT validation including signature verification |
| `/v1/admin/flag` | 403 Forbidden | Token accepted but authorization check failed        |

The `/v1` endpoint accepting the token without returning "INVALID TOKEN" suggested that it was not properly verifying the JWT signature. This meant the token's claims could potentially be modified without invalidation.

---

### Step 4 - JWT Token Manipulation
With the understanding that the `/v1` endpoint had weak token validation, the next step was to manipulate the JWT claims. The user's JWT token was decoded to reveal its structure:

```json
{
  "user_id": 123,
  "role": "user",
  "username": "testuser"
}
```

The following modifications were made to the token:
1. Changed `role` from `"user"` to `"admin"`
2. Changed `user_id` from the current user's ID to `1` (assuming the admin account would have the first ID)

The modified token was re-encoded (without a valid signature, since the endpoint wasn't verifying it).

---

### Step 5 - Exploiting the Vulnerability
The manipulated JWT token was used to make a request to the vulnerable endpoint:

1. Intercept a request to `/v1/admin/flag` in Burp Suite
2. Replace the Authorization header with the modified JWT token
3. Forward the request

The server accepted the manipulated token and returned the flag, confirming successful exploitation of:
- Missing JWT signature verification on legacy endpoints
- IDOR vulnerability allowing access to admin account via user ID manipulation

---

### Impact
- Complete bypass of authentication controls on legacy API endpoints
- Unauthorized access to administrative functionality
- Ability to impersonate any user by manipulating the user ID claim
- Potential for privilege escalation across the entire application
- Demonstrates how maintaining legacy endpoints without security parity creates critical vulnerabilities

---

### Vulnerability Classification
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Broken Authentication, JWT Signature Bypass
- **Secondary Issue:** IDOR (Insecure Direct Object Reference)
- **CWE:** CWE-287 - Improper Authentication
- **CWE:** CWE-345 - Insufficient Verification of Data Authenticity
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The application made several critical security mistakes:

1. **Inconsistent Security Controls:** The `/v1` API endpoints lacked the security improvements implemented in `/v2`, specifically JWT signature verification.

2. **Missing Token Signature Validation:** Legacy endpoints accepted JWT tokens without verifying the cryptographic signature, allowing attackers to forge claims.

3. **ID-Based Authorization:** The application relied solely on the `user_id` claim from the token for authorization decisions without additional verification, enabling IDOR attacks.

4. **Legacy Endpoint Exposure:** Outdated API versions remained accessible in production without proper deprecation or equivalent security controls.

---

### Remediation
- Implement consistent security controls across ALL API versions
- Properly deprecate and remove legacy API endpoints that cannot be secured
- Always verify JWT signatures using strong cryptographic algorithms (RS256, ES256)
- Never trust claims from tokens without signature verification
- Implement defense in depth by validating user permissions server-side against a database, not just token claims
- Use API versioning strategies that don't expose vulnerable legacy code
- Conduct security reviews when maintaining multiple API versions
- Implement monitoring and alerting for access to deprecated endpoints
- Consider using API gateways to enforce consistent authentication policies across all versions
- Regular security audits of all exposed endpoints, including legacy versions

---
