---
layout: post
title:  "BugForge - Daily - Tanuki"
date:   2026-01-13 20:40
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki
><br/><b>Vulnerabilities Covered:</b>
<br/>
IDOR (Insecure Direct Object Reference)
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates an Insecure Direct Object Reference (IDOR) vulnerability in the Tanuki flashcard application's statistics API endpoint. After registering an account and being assigned a user ID, the application exposes a `/api/stats/{id}` endpoint that lacks proper authorization controls. By manipulating the numeric ID parameter, an attacker can access other users' statistics and sensitive data, including a hidden achievement flag. The vulnerability highlights the importance of implementing object-level access control rather than relying solely on authentication to protect user-specific resources.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Registration and Baseline Analysis

The first step is to register a new user account and analyze the registration request and response.

During registration, the server response reveals important information:
- The API endpoint `/api/register` is used for account creation
- The newly created user is assigned an `id` of 4, indicating at least 3 other users exist on the platform
- No role parameter is visible in the request or response, making role manipulation unlikely for this challenge

This initial reconnaissance provides valuable context about the application's user structure.

---

### Step 2 - Application Feature Review

After logging in, the dashboard presents several features:
- A welcome message displaying the user's name
- Available study decks (e.g., "Linux Trivia")
- Navigation menu with: My Decks, Browse Decks, Stats, and Logout

With IDOR as the target vulnerability, the focus shifts to identifying endpoints that use predictable identifiers.

---

### Step 3 - Initial Endpoint Testing (Study Decks)

Clicking "Start Studying" on a deck loads the URL `/study/2`. The numeric identifier is an immediate candidate for IDOR testing.

Testing revealed:
- `/study/1` returns a "Planets & Moons" deck
- `/study/2` returns the "Linux Trivia" deck
- `/study/3` returns another valid deck

Using Burp Intruder to enumerate IDs 1-100 (with `limit=100` parameter) identified only 3 valid decks. Filtering responses for the flag pattern `bug{` yielded no results.

---

### Step 4 - Secondary Endpoint Testing (Decks API)

The `/decks/{id}` endpoint was also identified in the HTTP traffic. Similar enumeration was performed:
- Only 3 valid deck IDs were found
- No flag pattern was present in any response
- Testing `/decks/0` and `/study/0` returned no results (ruling out zero-indexed entries)

---

### Step 5 - Discovering the Hidden API Endpoint

After exhausting the obvious endpoints, the key lesson emerged: **always monitor HTTP traffic in the proxy, not just browser URLs**.

When navigating to the **Stats** page, the browser URL showed a static page with no identifiable parameters. However, inspecting the HTTP traffic revealed an API call:

```
GET /api/stats/4
```

The `4` in this endpoint correlates directly to the user ID assigned during registration.

---

### Step 6 - Exploiting the IDOR Vulnerability

With the `/api/stats/{id}` endpoint identified, IDOR testing was straightforward:

1. Sent the request to Burp Repeater
2. Changed the ID from `4` to `1`
3. Submitted the modified request

The server returned the statistics for user ID 1 without any authorization verification.

---

### Step 7 - Flag Retrieval

The response for `/api/stats/1` contained the flag embedded as an achievement field:

```json
{
  "achievement_flag": "bug{...}"
}
```

The challenge was completed by accessing another user's data through unauthorized parameter manipulation.

---

### Impact
- Unauthorized access to any user's statistics and personal data
- Exposure of sensitive achievement data and potentially other user information
- Complete breakdown of horizontal access controls
- Demonstrates how hidden API endpoints can expose IDOR vulnerabilities that aren't visible in the browser URL

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** User statistics API endpoint
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause

The `/api/stats/{id}` endpoint performs authentication (verifying the user is logged in) but fails to implement authorization (verifying the user has permission to access the requested resource). The application trusts the user-supplied ID parameter without validating that the authenticated user owns or has permission to view that data.

---

### Remediation
- Implement object-level authorization checks on all API endpoints that access user-specific data
- Use session-based user identification rather than user-supplied IDs where possible
- If user IDs must be exposed, use non-sequential UUIDs instead of predictable integers
- Log and monitor for suspicious patterns of sequential ID access
- Apply the principle of least privilege: users should only access their own resources by default

---

### Key Takeaways

1. **Always monitor HTTP traffic** - API calls may not be visible in the browser URL, especially in Single Page Applications (SPAs)
2. **Registration responses contain gold** - User IDs, role assignments, and API patterns revealed during registration are invaluable for reconnaissance
3. **Don't stop at the first endpoint** - Multiple endpoints may use identifiers, but only some may be vulnerable
4. **Predictable IDs are a red flag** - Sequential numeric identifiers (1, 2, 3, 4...) are prime candidates for IDOR testing

---
