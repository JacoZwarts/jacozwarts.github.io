---
layout: post
title:  "BugForge - Daily - Cheesy Does it"
date:   2026-01-12 20:40
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does it
><br/><b>Vulnerabilities Covered:</b>
<br/>
Business Logic Flaw
<br/>
<br/>
<b>Summary:</b>
<br/>
After creating an account and placing a normal pizza order, the checkout request was intercepted to analyze how pricing data is handled by the backend. The POST /order request was found to trust client-supplied unit and total prices, allowing these values to be modified to zero before submission. The server accepted the manipulated request and processed the order at $0, confirming a business logic flaw caused by missing server-side price validation.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation
Create a new user account and log in to the application.

---

### Step 2 - Place a Pizza Order
Browse the menu and place a pizza order using the standard checkout process.

---

### Step 3 - Intercept the Order Request
Intercept the checkout request and observe the `POST /order` request sent to the backend.

Notice that the request includes client-controlled pricing fields, such as:
- Unit price
- Total price


![Request to place order](/images/bug-forge/daily/cheesy-does-it/broken-logic/request-order-pizza.png)

---

### Step 4 - Modify Pricing Parameters
Modify the intercepted request by changing the pricing values:
- Set the unit price of the pizza to `0`
- Set the total order price to `0`

![Request to place order](/images/bug-forge/daily/cheesy-does-it/broken-logic/modified-order-pizza-request.png)


---

### Step 5 - Submit the Modified Request
Forward the modified `POST /order` request to the server.

---

### Step 6 - Verify the Outcome
Confirm that the order is successfully processed and accepted with a total cost of `$0`, demonstrating that the backend trusts client-supplied pricing data.

![Flag](/images/bug-forge/daily/cheesy-does-it/broken-logic/flag.png)

---


### Impact
- Ability to purchase items for free by manipulating client-side pricing
- Direct financial loss and revenue manipulation
- Undermines trust in the checkout and billing process

---

### Vulnerability Classification
- **OWASP Top 10:** Insecure Design
- **Vulnerability Type:** Business Logic Flaw (Client-Side Price Manipulation)
- **CWE:** CWE-602 - Client-Side Enforcement of Server-Side Security

---

### Root Cause
The backend trusts client-supplied pricing values (unit price and total) instead of calculating and validating prices server-side using authoritative product data.

---

### Remediation
- Recalculate all pricing server-side based on trusted product and pricing sources
- Ignore or strictly validate any client-supplied price fields
- Implement order integrity checks before payment processing
- Add monitoring and alerting for abnormal pricing or zero-value orders

---