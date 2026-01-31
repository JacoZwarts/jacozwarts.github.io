---
layout: post
title:  "BugForge - Daily - Sokudo"
date:   2026-01-01 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Access Control
<br/>
<br/>
<b>Summary:</b>
<br/>
Exploited **broken access control** via **HTTP verb tampering** on the `/api/stats` endpoint—POST was blocked (404) but PUT lacked authorization, allowing stats manipulation and flag retrieval.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Overview and Baseline Creation
Sokudo is a speed typing application that allows users to complete typing tests and track their performance statistics. The application features a typing test interface, a statistics dashboard, and a leaderboard.

After logging in, a typing session was completed to generate test data. This establishes a baseline and populates the statistics endpoint with data that can be used for further testing.

---

### Step 2 - Endpoint Discovery and Analysis
Using a proxy tool, the `/api/stats` endpoint was identified as the API responsible for retrieving the authenticated user's typing performance metrics. The GET request returns statistics from the most recent typing session including metrics like words per minute (WPM).

The response structure includes user-specific data, suggesting the endpoint may be vulnerable to manipulation if authorization checks are not consistently applied across all HTTP methods.

---

### Step 3 - HTTP Verb Tampering Attempt with POST
The first test involved changing the HTTP method from GET to POST. The request was sent to Replay, and the following modifications were made:

1. Changed the HTTP method from GET to POST
2. Copied the JSON response body from the GET request and added it as the POST request body
3. Updated the `Content-Type` header to `application/json` (critical step - the default URL-encoded content type will not work)

The server responded with a `404 Not Found` error indicating that the POST method is not supported on this endpoint, suggesting some level of method restriction is in place.

---

### Step 4 - HTTP Verb Tampering with PUT Method
Since POST was blocked, the PUT method was tested using the same approach:

1. Changed the HTTP method to PUT
2. Used the same JSON payload with modified statistics (e.g., changing WPM to 1000)
3. Ensured `Content-Type: application/json` header was set

The PUT request succeeded and returned the flag, confirming that while POST is restricted, the PUT method lacks proper authorization controls. This inconsistent enforcement of access controls across HTTP methods represents a critical vulnerability.

---

### Impact
- Unauthorized modification of user statistics and performance data
- Potential to manipulate leaderboard rankings through falsified statistics
- Demonstrates a complete bypass of access controls through HTTP method switching
- Indicates inconsistent security enforcement that may affect other endpoints
- Undermines the integrity of the typing test platform's competitive features

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control
- **Vulnerability Type:** HTTP Verb Tampering (HTTP Method Override)
- **Attack Surface:** API endpoint with inconsistent HTTP method authorization
- **CWE:** CWE-650 - Trusting HTTP Permission Methods on the Server Side

---

### Root Cause
The backend implemented access restrictions for the POST method on the `/api/stats` endpoint but failed to apply equivalent authorization controls to the PUT method. This inconsistency allowed attackers to bypass access restrictions by simply changing the HTTP verb. The server trusted the HTTP method context without enforcing uniform authorization logic across all supported verbs.

---

### Remediation
- Enforce consistent authorization checks across all HTTP methods for every endpoint
- Explicitly disable unsupported HTTP methods and return appropriate `405 Method Not Allowed` responses
- Implement centralized middleware or decorators that enforce authorization policies uniformly regardless of HTTP verb
- Use allow-lists to define which HTTP methods are permitted for each endpoint
- Validate that the authenticated user is authorized to perform the requested action on the referenced resource
- Conduct security testing that includes verb tampering scenarios across all API routes
- Log and monitor for unusual HTTP method usage patterns that may indicate exploitation attempts
- Apply the principle of least privilege - deny by default unless explicitly permitted

---
