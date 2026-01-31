---
layout: post
title:  "BugForge - Daily - Cheesy Does It"
date:   2025-12-29 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
IDOR (Insecure Direct Object Reference)
<br/>
<br/>
<b>Summary:</b>
<br/>
This vulnerability is an **Insecure Direct Object Reference (IDOR)** caused by missing server-side authorization checks when accessing order data. The application exposes predictable, sequential `order_id` values and trusts the client-supplied identifier without verifying that the authenticated user owns the referenced order. By enumerating and manipulating the `order_id` parameter, an attacker can access other users’ orders, resulting in horizontal privilege escalation and exposure of sensitive data.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Initial Account Creation and Order Placement

A new user account was created and a pizza order was placed through the application.  
After submitting the order, the response was reviewed to understand how order data is referenced and retrieved.

---

### Step 2 - Order ID Observation

The response returned an `order_id` value of `1`, indicating that order identifiers are assigned sequentially.

![Order request](/images/bug-forge/daily/cheesy-does-it/order-idor/order-request.png)

This suggested that order identifiers might be predictable and potentially enumerable.

---

### Step 3 - Automated Enumeration Testing

Using **Caido Automate**, the `order_id` parameter was systematically iterated from `1` to `100` to test how the backend responds to different values.

![Caido Automate](/images/bug-forge/daily/cheesy-does-it/order-idor/caido-automate-to-brute-force-idor.png)

The goal was to determine whether the application properly validated access to order data based on the authenticated user.

---

### Step 4 - Enumeration Results Analysis

During enumeration, only requests with `order_id=1` returned a valid response.  
All other values resulted in empty or invalid responses.

This confirmed that the backend exposes order data directly based on the supplied identifier.

---

### Step 5 - Second Account Creation

A second user account was created and another pizza order was placed.  
This allowed verification of how order identifiers are assigned across different users.

---

### Step 6 - Order ID Correlation

The newly created order was assigned `order_id=2`, confirming that order identifiers increment globally rather than being scoped to individual users.

This reinforced the hypothesis that access control checks were missing or improperly implemented.

---

### Step 7 - Manual Parameter Manipulation

The order retrieval request for the second user was manually modified by changing `order_id=2` to `order_id=1`.

This modification was performed directly in the request before sending it to the server.

---

### Step 8 - Unauthorized Order Access and Flag Retrieval

After sending the modified request, the application returned the first user’s order data.  
This confirmed that no authorization checks were enforced to ensure users could only access their own orders.

The unauthorized response revealed the flag, completing the challenge.

![Flag](/images/bug-forge/daily/cheesy-does-it/order-idor/idor-with-flag.png)

---

### Impact
- Unauthorized access to other users’ order information  
- Exposure of sensitive order details across accounts  
- Enables horizontal privilege escalation  
- Undermines trust in user data isolation  
- Demonstrates a classic access control failure that is trivial to exploit  

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control  
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)  
- **Attack Surface:** Order retrieval API  
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key  

---

### Root Cause

The backend relies solely on the client-supplied `order_id` to retrieve order data and does not verify whether the authenticated user is authorized to access the referenced order.  
Order identifiers are sequential and predictable, making them easy to enumerate and abuse across user accounts.

---

### Remediation
- Enforce strict server-side authorization checks to ensure users can only access their own orders  
- Avoid exposing sequential or predictable identifiers in client-facing APIs  
- Scope order queries to the authenticated user rather than trusting request parameters  
- Implement access control testing as part of the SDLC  
- Add monitoring to detect abnormal access patterns across order identifiers 

---