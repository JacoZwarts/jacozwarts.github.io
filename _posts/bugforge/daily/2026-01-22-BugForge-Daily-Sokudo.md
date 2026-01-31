---
layout: post
title:  "BugForge - Daily - Sokudo"
date:   2026-01-22 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,http-verb-tampering]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Access Control
HTTP Verb Tampering
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates a `broken access control` vulnerability exploited through `HTTP verb tampering` on a typing test statistics endpoint. After establishing a baseline during account registration, testing focused on the `/api/stats` endpoint which was designed to retrieve the authenticated user's typing performance data via GET requests. The endpoint accepted a `user_id` parameter but failed to properly validate user authorization across different HTTP methods. By leveraging Caido's HTTP verb toggle functionality and switching from GET to PUT while manipulating the `user_id` parameter, it was possible to access another user's statistics without authorization. This demonstrates how improper HTTP method handling combined with missing authorization checks creates a critical access control bypass, allowing attackers to view or modify resources belonging to other users by simply changing the HTTP verb used in the request.
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

![Registration Request & Response](/images/bug-forge/daily/Sokudo/broken-access-control-stats/register-request-response.png)

---

### Step 2 - Functionality Analysis
After logging in, it was observed that the application provides limited functionality: a typing test feature and a statistics dashboard. Several typing sessions were completed to populate the system with test data.

During endpoint analysis in Caido, the `/api/stats` endpoint was identified as returning the currently authenticated user's typing performance metrics. The response included a `user_id` field, which suggested that this identifier might be manipulated to access other users' data.

This raised the possibility of testing for `HTTP verb tampering`, a technique where different HTTP methods (GET, POST, PUT, DELETE, PATCH) are used to bypass access controls or trigger unexpected server-side behavior. Many applications implement authorization checks only for specific HTTP verbs while leaving others unprotected, creating opportunities for access control bypass.

---

### Step 3 - HTTP Verb Tampering on Stats Endpoint
Using Caido's Toggle GET/POST feature, the request method was changed from GET to POST. When toggling HTTP methods, it is critical to update the `Content-Type` header to `application/json` to ensure the server correctly parses the request body.

The JSON payload from the original GET response was used as the POST request body, and the `user_id` parameter was modified to `1` in an attempt to access another user's statistics.

![Caido Post Toggle](/images/bug-forge/daily/Sokudo/broken-access-control-stats/caido-toggle-post.png)

The server responded with a `404 Not Found` error and the message `Cannot POST /api/stats`, indicating that the endpoint does not support POST requests.

![POST - Failure](/images/bug-forge/daily/Sokudo/broken-access-control-stats/stats-post-request-failed.png)

Next, the PUT method was tested using the same approach. This time, the request was successful and returned the flag, confirming that the PUT method was implemented without proper authorization checks.

![PUT - Success](/images/bug-forge/daily/Sokudo/broken-access-control-stats/flag.png)

---

### Impact
- Unauthorized access to other users' typing statistics and performance data
- Potential to modify or manipulate statistics for arbitrary users
- Demonstrates a complete bypass of user-level authorization through HTTP method switching
- Exposes sensitive user activity and behavioral metrics
- Indicates a pattern of inconsistent authorization enforcement across HTTP verbs that may affect other endpoints

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control
- **Vulnerability Type:** HTTP Verb Tampering (HTTP Method Override)
- **Attack Surface:** API endpoint accepting multiple HTTP methods without consistent authorization
- **CWE:** CWE-650 - Trusting HTTP Permission Methods on the Server Side

---

### Root Cause
The backend implemented authorization checks for the GET method on the `/api/stats` endpoint but failed to apply the same controls to the PUT method. This inconsistency allowed attackers to bypass access restrictions by simply changing the HTTP verb while supplying a manipulated `user_id` parameter. The server trusted the HTTP method context without enforcing uniform authorization logic across all supported verbs.

---

### Remediation
- Enforce consistent authorization checks across all HTTP methods for every endpoint
- Validate that the authenticated user is authorized to access or modify the referenced resource, regardless of the HTTP verb used
- Explicitly disable unsupported HTTP methods and return appropriate `405 Method Not Allowed` responses
- Implement centralized middleware or decorators that enforce authorization policies uniformly
- Use allow-lists to define which HTTP methods are permitted for each endpoint
- Conduct security testing that includes verb tampering scenarios across all API routes
- Log and monitor for unusual HTTP method usage patterns that may indicate exploitation attempts

---