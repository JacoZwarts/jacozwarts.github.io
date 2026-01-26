---
layout: post
title:  "BugForge - Daily - Shady Oaks Finance"
date:   2026-01-02 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,parameter-tampering,insecure-design]
categories: [BugForge]
---

# Daily - Shady Oaks Finance
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Access Control
<br/>
Parameter Tampering
<br/>
Insecure Design
<br/>
<br/>
<b>Summary:</b>
<br/>
This vulnerability is a **broken access control issue caused by insecure design**, where the application trusts client-supplied input to set sensitive user attributes such as role assignments. Instead of handling role elevation logic server-side based on business rules, the application accepts a `role` parameter directly from the client during the upgrade process. By intercepting and modifying the upgrade request to change the role value from a standard user to `administrator`, an attacker can escalate privileges and gain unauthorized access to administrative functionality. This demonstrates a **fundamental design flaw** where authorization decisions are delegated to the client rather than being enforced by the backend.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Registration and Application Review

A new user account was registered and authenticated to the financial trading application.
After logging in, the application was explored to understand available functionality and identify potential privilege escalation vectors.

The application provided standard features including stock trading values, portfolio management, money exchange functionality, and trading history.

---

### Step 2 - Identifying the Upgrade Feature

An "Upgrade" feature was discovered that allowed users to start a 7-day free trial of premium trading insights.
This feature appeared to modify user permissions or roles to grant access to additional functionality.

Clicking the "Start 7-Day Free Trial" button changed the interface to display "Insider Tips", indicating that the user's role or permissions had been modified.

---

### Step 3 - Intercepting the Upgrade Request

The upgrade request was intercepted using a proxy tool to analyze the request structure and parameters being sent to the backend.

The request was identified as a POST request to the `/api/upgrade` endpoint.

---

### Step 4 - Analyzing Request Parameters

Upon inspecting the request body, a `role` parameter was discovered that was being sent from the client to the server.

This indicated a potential insecure design flaw where the application was trusting client-supplied input to set the user's role, rather than handling role assignment logic exclusively on the backend.

---

### Step 5 - Role Parameter Manipulation

The `role` parameter value in the upgrade request was modified from its original value to `administrator` before sending the request to the server.

This manipulation tested whether the backend would accept arbitrary role values supplied by the client.

---

### Step 6 - JWT Token Analysis

After sending the modified request, the JWT token returned in the response was analyzed using a JWT decoder.

The decoded token revealed that the `role` claim within the JWT now reflected the value `administrator`, confirming that the backend accepted the client-supplied role parameter without validation.

---

### Step 7 - Token Refresh and Admin Access

To ensure the modified JWT token was active in the browser session, the page was refreshed (alternatively, logging out and logging back in achieves the same result).

Upon refresh, an **Admin Panel** became accessible in the application interface, confirming successful privilege escalation.

---

### Step 8 - Flag Retrieval

Navigating to the Admin Panel revealed administrative functionality that should not be accessible to standard users.

Accessing the admin area returned the flag, completing the challenge.

---

### Impact
- Unauthorized privilege escalation to administrative roles
- Complete compromise of authorization controls through parameter tampering
- Access to sensitive administrative functionality and data
- Demonstrates a critical design flaw where role assignment is controlled by the client
- Enables full application compromise by any authenticated user

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control / Security Misconfiguration
- **Vulnerability Type:** Parameter Tampering / Insecure Design / Privilege Escalation
- **Attack Surface:** Upgrade API endpoint
- **CWE:** CWE-639 – Authorization Bypass Through User-Controlled Key

---

### Root Cause

The backend trusts client-supplied input to set sensitive user attributes, specifically the `role` parameter during the upgrade process.
Role assignment and privilege elevation should be handled exclusively on the backend based on business logic, user state, and authorization rules—not by accepting arbitrary values from client requests.

The application sends a role parameter in the request body (e.g., `{"role": "premium"}`) and allows the client to specify any value, including `administrator`, without server-side validation or authorization checks.

---

### Remediation

**Secure Design Pattern:**
Instead of accepting a `role` parameter from the client, the API should accept a business action parameter such as:
```json
{
  "confirm_free_trial": true
}
```

The backend should then handle role assignment based on:
- The authenticated user's identity (from JWT or session)
- Business rules (e.g., free trial eligibility, subscription status)
- Server-side authorization logic

**Additional Controls:**
- Never trust client-supplied input for security-critical decisions such as role or permission assignment
- Implement strict server-side validation and authorization checks on all privilege-modifying endpoints
- Handle role assignment exclusively on the backend based on authenticated user identity and business logic
- Reject or ignore any client-supplied role or permission parameters
- Use strongly typed business actions (e.g., "start_trial", "upgrade_subscription") rather than exposing internal role values
- Add security testing for parameter tampering and privilege escalation scenarios
- Implement comprehensive logging and monitoring for role changes and privilege escalations

---

### Vulnerability Summary

This vulnerability represents a **fundamental insecure design flaw** where the application delegates authorization decisions to the client rather than enforcing them server-side. By trusting a client-supplied `role` parameter in the upgrade request, the application allows any authenticated user to escalate their privileges to administrator simply by tampering with the request. The proper design requires the backend to handle all role assignment logic based on business rules and user state, accepting only high-level business actions from the client (such as "start trial") while maintaining complete control over authorization decisions.

---