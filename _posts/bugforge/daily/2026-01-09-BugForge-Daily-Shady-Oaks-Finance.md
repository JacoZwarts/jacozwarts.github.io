---
layout: post
title:  "BugForge - Daily - Shady Oaks Finance"
date:   2026-01-09 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [jwt,authentication-bypass,none-algorithm,broken-authentication]
categories: [BugForge,daily,shady-oaks-finance]
---

# Daily - Shady Oaks Finance
><br/><b>Vulnerabilities Covered:</b>
<br/>
JWT None Algorithm Attack
<br/>
Broken Authentication
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates a **JWT (JSON Web Token) authentication bypass** vulnerability caused by improper algorithm validation. The application accepts JWTs with the `alg` header set to `none`, which instructs the server to skip signature verification entirely. By modifying the JWT header to use the none algorithm and removing the signature portion, an attacker can tamper with the payload claims—specifically changing the `role` from `user` to `admin`—without needing to know the secret key used for signing. This allows complete authentication bypass and privilege escalation to administrative access, enabling retrieval of sensitive data from protected admin endpoints.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Registration and JWT Discovery

Register a new account on the Shady Oaks Financial trading platform. After successful registration, the application authenticates the user and returns a JWT token in the response.

Intercept the registration request to `/api/register` using a proxy tool such as Burp Suite. The response contains a JWT token that will be used for all subsequent authenticated requests via the `Authorization: Bearer <token>` header.

---

### Step 2 - JWT Structure Analysis

Analyze the JWT token structure. A JWT consists of three Base64-encoded parts separated by periods:
- **Header** - Contains the algorithm (`alg`) and token type (`typ`)
- **Payload** - Contains the claims (user data such as `id`, `username`, `role`)
- **Signature** - Cryptographic signature to verify token integrity

Using a JWT decoder (such as jwt.io, Burp Suite's JWT Editor extension, or browser DevTools), decode the token to reveal:

**Header:**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:**
```json
{
  "id": 4,
  "username": "tester",
  "role": "user",
  "iat": 1767932707
}
```

The `HS256` algorithm indicates a symmetric signing algorithm, and the `role` claim set to `user` is the target for privilege escalation.

---

### Step 3 - Testing Signature Validation

Before attempting JWT attacks, verify whether the server actually validates the token signature.

Modify the `role` claim from `user` to `admin` in the payload while keeping the original signature. Send a request with this tampered token. The server responds with an "Invalid token" error, confirming that signature validation is being performed.

This rules out the simple approach of modifying the payload without addressing the signature.

---

### Step 4 - None Algorithm Attack

The none algorithm attack exploits servers that accept JWTs with `alg` set to `none`, effectively disabling signature verification.

Modify the JWT:
1. Change the header `alg` value from `HS256` to `none`
2. Update the payload `role` from `user` to `admin`
3. Remove the signature portion entirely, but **keep the trailing period**

The modified token structure becomes:
```
<base64-header>.<base64-payload>.
```

Using a tool like `jwt_tool`:
```bash
python3 jwt_tool.py "<original-jwt>" -X a
```

Or manually craft the token by Base64-encoding the modified header and payload.

**Important:** The trailing period after the payload is required for the token to be valid.

---

### Step 5 - Validating the Attack

Send a request to a protected endpoint (such as `/api/stocks` or `/api/verify-token`) with the modified JWT in the `Authorization` header.

If the server accepts the token and returns a valid response instead of an authentication error, the none algorithm attack is successful. This confirms that the server does not properly enforce signature verification when the algorithm is set to none.

---

### Step 6 - Admin Endpoint Discovery

Identify administrative endpoints by examining the client-side JavaScript files. Using browser DevTools (Debugger tab) or a bookmarklet to extract endpoints from JavaScript:

Navigate to the JavaScript files and search for API routes. Look for files like `AdminPanel.js` which reveal protected endpoints:
- `/api/admin/users`
- `/api/admin/flag`

These endpoints require administrative privileges to access.

---

### Step 7 - Flag Retrieval

With the forged admin JWT, access the `/api/admin/flag` endpoint.

Add the `Authorization` header with the Bearer token containing the modified JWT (with `alg: none` and `role: admin`):

```
Authorization: Bearer <forged-jwt-token>
```

The server accepts the forged token and returns the flag, confirming successful privilege escalation through JWT tampering.

---

### Impact
- Complete authentication bypass through algorithm manipulation
- Unauthorized privilege escalation from user to administrator
- Access to sensitive administrative functionality and data
- Ability to impersonate any user by modifying JWT claims
- Full compromise of application authorization controls

---

### Vulnerability Classification
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** JWT None Algorithm Attack / Broken Authentication
- **Attack Surface:** Authentication token handling
- **CWE:** CWE-287 - Improper Authentication
- **CWE:** CWE-345 - Insufficient Verification of Data Authenticity

---

### Root Cause

The backend JWT library is configured to accept the `none` algorithm, which instructs the server to skip signature verification entirely. When the server receives a token with `alg: none`, it processes the payload claims without validating the signature, allowing attackers to forge tokens with arbitrary claims.

This vulnerability typically occurs when:
- JWT libraries have the none algorithm enabled by default
- Explicit algorithm validation is not enforced during token verification
- The server trusts the `alg` claim from the token header rather than using a server-side whitelist

---

### Remediation

**Algorithm Enforcement:**
- Explicitly specify allowed algorithms on the server side during token verification
- Never trust the `alg` claim from the token header
- Disable or reject the `none` algorithm in all JWT library configurations

**Example (Node.js with jsonwebtoken):**
```javascript
jwt.verify(token, secretKey, { algorithms: ['HS256'] });
```

**Additional Controls:**
- Use asymmetric algorithms (RS256, ES256) for enhanced security where key distribution is feasible
- Implement token expiration (`exp` claim) to limit the window for token abuse
- Store sensitive authorization data server-side rather than in JWT claims
- Implement token revocation mechanisms for compromised tokens
- Add comprehensive logging for authentication failures and anomalous token patterns
- Regularly audit JWT library configurations and update dependencies
- Consider using a well-tested authentication framework that handles JWT security by default

---

### Tools Used
- Burp Suite with JWT Editor extension
- jwt_tool (`python3 jwt_tool.py -X a` for none algorithm attack)
- Browser DevTools for JavaScript analysis and Local Storage inspection
- JWT decoders (jwt.io, JWT Auditor)

---
