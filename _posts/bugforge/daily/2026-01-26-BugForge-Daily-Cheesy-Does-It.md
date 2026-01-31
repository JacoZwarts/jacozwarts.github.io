---
layout: post
title:  "BugForge - Daily - Cheesy Does It"
date:   2026-01-26 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw,insufficient-validation]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
Business Logic Flaw<br/>
Insufficient Input Validation
<br/>
<br/>
<b>Summary:</b>
<br/>
A business logic flaw in the refund endpoint allows arbitrary refund amounts without validation against actual order values. The API endpoint and payload structure were discovered through client-side JavaScript analysis (`OrderTracking.js`), enabling fraudulent refund requests that bypass all server-side checks.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
Begin by exploring the core functionality of the pizza ordering application. Navigate through the ordering process including menu selection, cart management, and checkout to understand the standard user flow. Identify all available features such as order tracking, order history, and any post-purchase functionality like reporting issues or requesting refunds.

---

### Step 2 - Source Code Analysis
While reviewing the application, inspect the client-side JavaScript files for hints about API endpoints and business logic. The `OrderTracking.js` file reveals a `refundResult.flag` variable, indicating that the challenge flag is associated with the refund functionality.

![Order Tracking - Refund Source Code](/images/bug-forge/daily/cheesy-does-it/business-logic-flaw-refund/OrderTracking-Refund-Flag-Code.png)

---

### Step 3 - API Endpoint Discovery
Further analysis of the `OrderTracking.js` source code reveals the `handleSubmitReport` function which exposes the refund API endpoint and the expected payload structure. This includes the endpoint URL and the parameters required to submit a refund request.

![Order Tracking - Refund Payload](/images/bug-forge/daily/cheesy-does-it/business-logic-flaw-refund/OrderTracking-Refund-Post-Request.png)

---

### Step 4 - Exploiting the Refund Endpoint
Using the discovered endpoint and payload structure, craft a malicious refund request with an arbitrary amount of $1000. Submit the request to the refund API endpoint. The server processes the fraudulent refund without validating whether the amount corresponds to any legitimate order, returning the challenge flag and confirming the vulnerability.

![Flag](/images/bug-forge/daily/cheesy-does-it/business-logic-flaw-refund/flag.png)

---

### Impact
- Unauthorized refunds for arbitrary amounts unrelated to actual purchases
- Direct financial loss for the organization through fraudulent refund claims
- Ability to drain company funds by repeatedly exploiting the refund endpoint
- Complete bypass of legitimate refund validation controls
- Potential for automated exploitation at scale using scripted requests

---

### Vulnerability Classification
- **OWASP Top 10:** A04:2021 - Insecure Design (lack of server-side validation for financial operations)
- **Vulnerability Type:** Business Logic Flaw / Insufficient Input Validation
- **Attack Surface:** Refund processing API endpoint
- **CWE:** CWE-840 - Business Logic Errors

---

### Root Cause
The backend fails to implement proper validation when processing refund requests. The application does not verify that the requested refund amount matches an actual order total, nor does it confirm that the user is authorized to request a refund for a specific order. The server implicitly trusts the client-supplied refund amount without cross-referencing it against order records, allowing attackers to request refunds for arbitrary amounts that were never paid.

---

### Remediation
- Validate all refund requests against the original order amount stored in the database
- Ensure refund amounts cannot exceed the total value of the associated order
- Implement authorization checks to verify the user requesting the refund is the original purchaser
- Require a valid order ID for all refund requests and validate its existence and ownership
- Add server-side rate limiting on refund endpoints to prevent automated abuse
- Implement approval workflows for refunds above configurable thresholds
- Log all refund requests with detailed audit trails for fraud detection and investigation
- Never trust client-supplied monetary values; always recalculate amounts server-side

---