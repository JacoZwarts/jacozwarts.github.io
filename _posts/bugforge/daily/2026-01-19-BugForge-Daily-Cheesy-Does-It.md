---
layout: post
title:  "BugForge - Daily - Cheesy Does It"
date:   2026-01-19 20:02
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw,type-confusion]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
 Business Logic Flaw<br/>
 Type Confusion
<br/>
<br/>
<b>Summary:</b>
<br/>
This vulnerability is a business logic flaw in the checkout process where the backend accepts client-supplied discount data without enforcing strict validation or business rules. Although individual discount codes are validated, the application fails to handle **`array-based`** input correctly, allowing an attacker to submit multiple discount codes in a single request and receive cumulative discounts. This results in unauthorized price reductions, potential financial loss, and abuse of promotional mechanisms, highlighting weak server-side enforcement of pricing logic and over-trust in client-controlled request structures.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation & Baseline Review
Begin by inspecting the account registration flow. Pay close attention to the request and response to identify whether any role, permission, or privilege-related fields are being assigned or trusted during account creation.

![User registration request](/images/bug-forge/daily/cheesy-does-it/business-logic-array-injection/registration-request-analysis.png)

---

### Step 2 - Identifying the Discount Hint
After logging in, review the dashboard for any informational messages that may influence application behavior. An alert is displayed stating:  
`Use the discount code PIZZA-10 for a 10% discount today only!`

This serves as a strong indicator to further investigate how discounts are handled.

![Dashboard discount hint](/images/bug-forge/daily/cheesy-does-it/business-logic-array-injection/dashboard-hint.png)

---

### Step 3 - Inspecting the Checkout Flow
Proceed to the checkout process and observe how pricing and discounts are applied from the UI. Capture and analyze the order submission request to understand how the discount code is processed server-side.

![Checkout Process UI](/images/bug-forge/daily/cheesy-does-it/business-logic-array-injection/checkout-ui.png)

![Order Request](/images/bug-forge/daily/cheesy-does-it/business-logic-array-injection/order-request.png)

---

### Step 4 - Applying Multiple Discount Codes
First, attempt to modify the discount code from `PIZZA-10` to `PIZZA-20`. This does not succeed, indicating server-side validation on individual values.

Next, change the discount code parameter to an array and submit multiple discount codes in a single request. This approach succeeds, resulting in multiple discounts being applied to the order.

![Discount Code Array Injection](/images/bug-forge/daily/cheesy-does-it/business-logic-array-injection/discount-code-array.png)

---

### Impact
- Unauthorized application of multiple discount codes in a single order  
- Financial loss due to unintended cumulative discounts  
- Circumvention of business rules intended to limit promotions  
- Undermines trust in pricing and promotional controls  
- Demonstrates a business logic flaw that can be abused at scale

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control  
- **Vulnerability Type:** Business Logic Flaw / Array Injection  
- **Attack Surface:** Checkout and order processing API  
- **CWE:** CWE-840 - Business Logic Errors  

---

### Root Cause
The backend fails to enforce strict validation on the `discountCode` parameter and implicitly trusts the client-provided data structure. While single discount values are validated, the API does not account for array-based input, allowing multiple discount codes to be processed without proper normalization or server-side enforcement of business rules.

---

### Remediation
- Enforce strict server-side validation on discount code inputs, including data type checks  
- Normalize and validate inputs to ensure only a single discount code can be applied per order  
- Explicitly reject array or unexpected data structures for promotional fields  
- Centralize discount logic on the server to prevent client-side manipulation  
- Add monitoring and alerting for anomalous discount usage patterns to detect abuse early

---