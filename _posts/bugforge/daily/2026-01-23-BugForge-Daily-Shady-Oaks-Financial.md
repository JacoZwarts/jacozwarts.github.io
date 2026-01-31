---
layout: post
title:  "BugForge - Daily - Shady Oaks Finance"
date:   2026-01-23 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [race-condition,toctou]
categories: [BugForge,daily,shady-oaks-finance]
---

# Daily - Shady Oaks Finance
><br/><b>Vulnerabilities Covered:</b>
<br/>
Race Condition
Concurrency Control Bypass
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates a race condition vulnerability in a currency exchange endpoint where balance verification and balance deduction are not performed as atomic operations. When multiple concurrent requests target the `/api/convert-currency` endpoint simultaneously, they all read the same account balance before any deductions are applied, allowing each request to proceed as if sufficient funds are available. This Time-of-Check Time-of-Use (TOCTOU) flaw occurs because the application lacks proper synchronization mechanisms such as database-level locking or transactional isolation, enabling an attacker to bypass balance validation checks and perform currency conversions far exceeding their actual available funds. This type of vulnerability can lead to unauthorized fund transfers, financial discrepancies, and platform liquidity drainage in real-world applications.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
Initial testing focused on core application functionality including trading features and profile management. Various requests were intercepted and analyzed for common vulnerabilities such as parameter manipulation, privilege escalation, and insecure direct object references. No significant vulnerabilities were identified during this phase.

---

### Step 2 - Currency Exchange Functionality Testing
Testing shifted to the currency conversion feature. Standard security tests were performed, including attempts to manipulate exchange rates, modify transaction amounts, and access other users' conversion history through IDOR techniques. The endpoint appeared to implement proper authorization and input validation for single requests.

However, the endpoint's behavior under concurrent load had not yet been examined. Race conditions occur when multiple operations execute simultaneously against shared resources without proper synchronization mechanisms. In financial applications, this typically manifests when balance checks and balance updates are not atomic operations, allowing multiple transactions to read the same balance before any writes occur.

To test for this vulnerability, the `/api/convert-currency` POST request was sent to Caido's Automate tool. A custom header `Test:§1§` was added with a placeholder marker, and the payload was configured to use sequential numbers from 1 to 1000.

![Caido - Payload configuration](/images/bug-forge/daily/shady-oaks-financial/race-condition/caido-automate-payload-configuration.png)

---

### Step 3 - Concurrent Request Exploitation
The Automate settings were configured to execute 50 concurrent workers, ensuring that multiple requests would be processed simultaneously by the server. This level of concurrency is designed to exploit the time window between balance verification and balance deduction operations.

![Caido - Settings - Concurrency](/images/bug-forge/daily/shady-oaks-financial/race-condition/caido-automate-payload-settings.png)

Upon analyzing the automation results, one of the responses returned the challenge flag. This confirms that the application failed to implement proper locking or transactional integrity, allowing concurrent requests to bypass balance constraints.

![Flag](/images/bug-forge/daily/shady-oaks-financial/race-condition/flag.png)

---

### Impact
- Unauthorized currency conversion beyond available account balance
- Financial discrepancies and loss of funds for the organization
- Ability to drain platform liquidity through repeated exploitation
- Complete bypass of transaction validation controls under concurrent load
- Potential for automated exploitation at scale using scripted concurrent requests

---

### Vulnerability Classification
- **OWASP Top 10:** A04:2021 - Insecure Design (lack of concurrency controls and transaction integrity)
- **Vulnerability Type:** Race Condition / Time-of-Check Time-of-Use (TOCTOU)
- **Attack Surface:** Currency conversion API endpoint
- **CWE:** CWE-362 - Concurrent Execution using Shared Resource with Improper Synchronization ('Race Condition')

---

### Root Cause
The backend fails to implement atomic operations or proper locking mechanisms when processing currency conversions. The vulnerability arises because balance verification and balance deduction occur as separate, non-atomic operations. When multiple requests arrive simultaneously, they all read the same account balance before any deductions are applied, allowing each request to proceed as if sufficient funds are available. This violates the ACID properties required for financial transactions, specifically atomicity and isolation.

---

### Remediation
- Implement database-level locking mechanisms such as pessimistic locking or optimistic locking with version control
- Use database transactions with appropriate isolation levels (e.g., SERIALIZABLE) to ensure atomic read-check-write operations
- Implement idempotency keys to prevent duplicate processing of the same logical transaction
- Add rate limiting and request throttling to reduce the window for concurrent exploitation
- Use queuing systems to serialize transaction processing for the same account
- Implement comprehensive transaction logging and monitoring to detect anomalous patterns of concurrent requests
- Add balance validation checks at both application and database constraint levels
- Consider implementing distributed locking mechanisms (e.g., Redis locks) in horizontally scaled environments
- Conduct load testing and concurrency testing as part of regular security assessments

---