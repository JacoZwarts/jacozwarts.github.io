---
layout: post
title:  "BugForge - Daily - Cafe Club"
date:   2025-12-28 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [brute-force,business-logic-flaw]
categories: [BugForge]
---

# Daily - Cafe Club
><br/><b>Vulnerabilities Covered:</b>
<br/>
Brute Force
<br/>
Business Logic Flaw
<br/>
<br/>
<b>Summary</b>
<br/>
This vulnerability is a `business logic flaw involving predictable identifiers and brute force`, where gift card codes are generated with insufficient entropy and only a small portion of the value changes between issuances. By identifying that only the final four digits varied, an attacker was able to `brute force` the gift card redemption endpoint to enumerate valid codes and redeem gift cards belonging to other users. The absence of ownership checks, rate limiting, and anti-automation controls enables this attack to scale, leading to unauthorized redemption, financial loss, and abuse of stored value functionality.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Functionality Review

The first step was to analyze the gift card purchase and redemption workflow to understand how gift cards are generated and validated.  
This included reviewing how codes are issued during purchase and how they are processed when submitted for redemption.

The objective was to identify which portions of the gift card code were meaningful and whether adequate server-side validation and ownership checks were enforced.

---

### Step 2 - Purchasing Gift Cards

Multiple gift cards were purchased through the application to collect a representative sample of issued codes.  
Each full gift card code was recorded for later comparison.

This established a baseline for assessing code structure and randomness.

---

### Step 3 - Identifying Predictable Patterns

By comparing the collected gift card codes, a clear pattern emerged.  
Most of the gift card code remained static or predictable, while only the final four letters changed between purchases.

This demonstrated insufficient entropy in the gift card generation process and indicated that the remaining keyspace was small enough to brute force.

---

### Step 4 - Wordlist Preparation

A wordlist was prepared to enumerate all possible combinations for the final four digits of the gift card code.  
AI was used to efficiently generate a complete wordlist covering the entire four-digit range.

This wordlist was later used to automate redemption attempts.

![Wordlist](/images/bug-forge/daily/cafe-club/brute-force/giftcard-wordlist.png)

---

### Step 5 - Redeem Endpoint Identification

The gift card redemption endpoint was identified by observing network requests made when submitting a gift card code through the application.  
This revealed the request structure and parameters used to validate gift card codes.

Understanding this endpoint was required to automate the attack.

---

### Step 6 - Brute Force Execution

Using the prepared wordlist, the redemption endpoint was systematically brute forced by varying only the final four digits of the gift card code.  
All other portions of the code remained unchanged, matching the predictable structure identified earlier.

Requests were sent iteratively until a valid gift card was accepted.

---

### Step 7 - Flag Retrieval

The attack succeeded once a valid gift card code was redeemed that did not belong to the authenticated user.  
This confirmed that gift cards were not properly bound to user accounts and that no effective anti-automation controls were in place.

Redeeming the unauthorized gift card revealed the flag and completed the challenge.

---

### Impact
- Unauthorized redemption of gift cards belonging to other users  
- Financial loss due to unauthorized use of stored monetary value  
- Enumeration and abuse of gift card balances at scale  
- Undermines trust in payment and gift card systems  
- Demonstrates a business logic flaw that can be automated for large-scale abuse  

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control  
- **Vulnerability Type:** Business Logic Flaw / Predictable Identifier  
- **Attack Surface:** Gift card redemption API  
- **CWE:** CWE-640 - Weak Password Recovery Mechanism for Forgotten Password (applied to weak token generation)  

---

### Root Cause

The backend generates gift card codes with insufficient randomness and relies on a partially static structure.  
Validation logic trusts the submitted gift card code without enforcing ownership checks or rate limiting, allowing attackers to brute force valid codes by enumerating the small remaining keyspace.

Additionally, the redemption endpoint lacks effective protections against automated abuse.

---

### Remediation
- Generate gift card codes using cryptographically secure random values with sufficient entropy  
- Bind gift cards to specific user accounts or enforce ownership checks during redemption  
- Implement strict rate limiting and anti-automation controls on redemption endpoints  
- Detect and block repeated failed redemption attempts  
- Monitor and alert on anomalous gift card redemption patterns  

---