---
layout: post
title:  "BugForge - Daily - Cafe Club"
date:   2026-01-25 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [race-condition,toctou]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club
><br/><b>Vulnerabilities Covered:</b>
<br/>
Race Condition
Time-of-Check Time-of-Use (TOCTOU)
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge exploits a TOCTOU (Time-of-Check Time-of-Use) race condition in the checkout flow where the cart is read twice without synchronization, once for price calculation and again for order fulfillment. By sending parallel requests (checkout + multiple add-to-cart) using Burp, items can be injected between these reads, resulting in receiving unpaid items.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis and Order Flow Mapping
Initial testing focused on understanding the standard order flow by completing a legitimate purchase. The checkout process was analyzed by intercepting all requests to identify the sequence of API calls involved in order placement.

![Order Flow - Endpoints](/images/bug-forge/daily/cafe-club/race-condition/endpoints-order-flow.png)

The checkout interface displays the cart contents and total price before payment confirmation.

![Order Flow - Checkout UI](/images/bug-forge/daily/cafe-club/race-condition/checkout-ui.png)

A successful order completion confirms that the standard flow functions as expected, establishing a baseline for subsequent testing.

![Order Flow - Endpoints](/images/bug-forge/daily/cafe-club/race-condition/order-successful.png)

---

### Step 2 - Race Condition Identification
Analysis of the checkout process revealed that the application reads the cart state twice during order processing: once for price calculation and validation, and again for order fulfillment. This separation between verification and execution creates a potential TOCTOU (Time-of-Check Time-of-Use) vulnerability. By sending multiple requests in parallel, it may be possible to modify the cart contents between these two reads.

---

### Step 3 - Parallel Request Exploitation
To exploit the race condition, the following attack flow was executed:

1. Add a single item to the cart via `POST /api/cart`

2. Intercept the checkout request (`POST /api/checkout`) and send it to Repeater

![Repeater - Checkout Request](/images/bug-forge/daily/cafe-club/race-condition/api-checkout-request-repeater.png)

3. Drop the intercepted checkout request to prevent premature execution

4. Capture multiple `POST /api/cart` requests (for different items) and send each to Repeater

![Repeater - Add Cart Request](/images/bug-forge/daily/cafe-club/race-condition/add-item-to-cart-repeater.png)

5. Group all requests together (1 checkout + 4 add-to-cart requests)

6. Send the group in parallel using Burpsuite's "Send in parallel" feature

When the requests hit the server simultaneously, the checkout request begins processing and reads the cart to calculate the total price. During the payment processing window, the add-to-cart requests execute and inject additional items into the cart. When checkout completes, it reads the cart again for order fulfillment but doesn't recalculate the price. The result: we're charged for one item but receive all items that were added during the race window.

![Group Request & Send group Parallel](/images/bug-forge/daily/cafe-club/race-condition/flag.png)

---

### Impact
- Unauthorized acquisition of products without payment
- Significant financial losses for the e-commerce platform
- Ability to obtain items at a fraction of their actual cost
- Complete bypass of payment validation controls under concurrent request conditions
- Potential for automated exploitation at scale to systematically exploit the ordering system
- Damage to business reputation and customer trust when discovered

---

### Vulnerability Classification
- **OWASP Top 10:** A04:2021 - Insecure Design (lack of atomic transaction processing and concurrency controls)
- **Vulnerability Type:** Race Condition / Time-of-Check Time-of-Use (TOCTOU)
- **Attack Surface:** E-commerce checkout and cart management API endpoints
- **CWE:** CWE-362 - Concurrent Execution using Shared Resource with Improper Synchronization ('Race Condition')

---

### Root Cause
The backend fails to implement atomic operations when processing checkout transactions. The vulnerability exists because the application reads the cart state at two separate points: first during price calculation and payment processing, then again during order fulfillment. These operations occur as separate, non-atomic database queries without proper locking mechanisms. When concurrent add-to-cart requests arrive during the processing window, they successfully modify the cart contents before the final order assembly, but after price verification has completed. This violates the ACID properties required for transactional integrity, specifically atomicity and isolation, allowing attackers to manipulate the cart state between verification and execution.

---

### Remediation
- Implement database-level locking on cart records during the entire checkout transaction
- Use database transactions with appropriate isolation levels (e.g., SERIALIZABLE) to ensure atomic read-process-write operations
- Create a snapshot of cart contents at checkout initiation and use only that snapshot for both pricing and fulfillment
- Implement optimistic locking with version control on cart records to detect concurrent modifications
- Add a final price recalculation step immediately before order creation within the same transaction
- Implement request rate limiting on cart modification endpoints during active checkout sessions
- Use queuing systems to serialize checkout processing for the same cart/session
- Add comprehensive transaction logging and monitoring to detect patterns of concurrent cart manipulation
- Lock the cart from modifications once checkout is initiated until completion or timeout
- Conduct regular concurrency and race condition testing as part of security assessments

---