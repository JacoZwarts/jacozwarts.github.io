---
layout: post
title:  "BugForge - Daily - Tanuki"
date:   2025-12-30 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [mass-assignment]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki
><br/><b>Vulnerabilities Covered:</b>
<br/>
Mass Assignment
<br/>
<br/>
<b>Summary:</b>
<br/>
This vulnerability is a `mass assignment-driven privilege escalation` where the application trusts client-supplied input during user registration and allows sensitive attributes, such as user_role, to be set directly by the user. By manipulating the registration request and changing the role from a standard user to an administrator, an attacker can create an account with elevated privileges. The absence of server-side allowlisting and role enforcement results in a complete breakdown of authorization controls and unrestricted access to administrative functionality.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Initial Account Registration and Application Review

A new user account was registered to establish a baseline understanding of the application.  
After logging in, the application was reviewed to identify available functionality, user-controlled inputs, and any areas where client-supplied data influenced backend behavior.

Special attention was paid to authentication- and authorization-related features.

---

### Step 2 - Baseline Authorization Testing

Standard authorization and IDOR testing was performed across the application.  
No direct object reference issues were identified through conventional parameter manipulation or endpoint testing.

This indicated that access control weaknesses, if present, might exist earlier in the user lifecycle.

---

### Step 3 - Secondary Account Creation

A second user account was registered to further analyze differences in behavior during account creation.  
The registration request payload was closely inspected to identify any fields that should not be controlled by the client.

---

### Step 4 - Identifying Mass Assignment Weakness

Reviewing the registration request revealed that the `user_role` parameter was included in the client-side payload and passed directly to the backend during account creation.

This indicated a potential mass assignment vulnerability, as role assignment should be handled exclusively server-side.

---

### Step 5 - Role Manipulation

Before submitting the registration request, the `user_role` value was modified from `user` to `admin`.

The manipulated request was then sent to the server without any client-side validation errors.

---

### Step 6 - Authentication with Elevated Privileges

The newly created account was used to authenticate to the application.  
Upon login, indicators of elevated privileges were observed, suggesting that the modified role value had been accepted by the backend.

---

### Step 7 - Unauthorized Admin Access and Flag Retrieval

An **Admin** tab was visible within the application interface.  
Navigating to the admin functionality confirmed unauthorized administrative access.

Accessing the admin area revealed the flag, completing the challenge.

---

### Impact
- Unauthorized privilege escalation to administrative roles  
- Full compromise of application integrity and security controls  
- Access to sensitive administrative functionality and data  
- Potential for account takeover, data manipulation, or service disruption  
- Demonstrates a critical authorization flaw exploitable during registration  

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control  
- **Vulnerability Type:** Mass Assignment / Privilege Escalation  
- **Attack Surface:** User registration API  
- **CWE:** CWE-915 - Improperly Controlled Modification of Dynamically-Determined Object Attributes  

---

### Root Cause

The backend implicitly trusts client-supplied input during account registration and allows sensitive attributes such as `user_role` to be set directly by the user.  
No server-side allowlist or enforcement is applied to restrict which fields may be modified during object creation, resulting in a mass assignment vulnerability.

---

### Remediation
- Enforce strict server-side allowlists for user-controllable fields during registration  
- Explicitly set sensitive attributes such as roles on the backend only  
- Reject or ignore unexpected parameters in request payloads  
- Separate user privilege assignment from user creation workflows  
- Add security testing for mass assignment and privilege escalation scenarios  

---

### Vulnerability Summary

This vulnerability is a **mass assignment flaw leading to privilege escalation**, where the application allows client-controlled input to define sensitive attributes during user registration. By modifying the `user_role` parameter in the registration request, an attacker can create an account with administrative privileges. The lack of server-side enforcement and attribute allowlisting enables full compromise of authorization controls and unrestricted access to administrative functionality.

---