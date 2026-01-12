---
layout: post
title:  "BugForge - Daily - Cheesy Does is - Broken Logic"
date:   2026-01-12 20:40
image:  /images/bug-forge/bugforge-logo.png
tags:   [Broken Logic]
categories: [BugForge]
---

# Daily - Cheesy Does it - Broken Logic
><b>Objective:</b>
<br/>
<b>Vulnerabilities Covered:</b>
<br/>
Broken Logic
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>

## Solution


### Step 1 – Account Creation
Create a new user account and log in to the application.

### Step 2 – Place a Pizza Order
Browse the menu and place a pizza order using the standard checkout process.

### Step 3 – Intercept the Order Request
Intercept the checkout request and observe the `POST /order` request sent to the backend.

Notice that the request includes client-controlled pricing fields, such as:
- Unit price
- Total price


![Request to place order](/images/bug-forge/daily/cheesy-does-it-broken-logic/request-order-pizza.png)

### Step 4 – Modify Pricing Parameters
Modify the intercepted request by changing the pricing values:
- Set the unit price of the pizza to `0`
- Set the total order price to `0`


![Request to place order](/images/bug-forge/daily/cheesy-does-it-broken-logic/modified-order-pizza-request.png)

### Step 5 – Submit the Modified Request
Forward the modified `POST /order` request to the server.

### Step 6 – Verify the Outcome
Confirm that the order is successfully processed and accepted with a total cost of `$0`, demonstrating that the backend trusts client-supplied pricing data.

![Flag](/images/bug-forge/daily/cheesy-does-it-broken-logic/flag.png)

---


