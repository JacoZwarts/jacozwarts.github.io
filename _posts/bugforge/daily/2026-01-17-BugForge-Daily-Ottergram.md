---
layout: post
title:  "BugForge - Daily - Ottergram"
date:   2026-01-17 19:25
image:  /images/bug-forge/bugforge-logo.png
tags:   [web-sockets,idor]
categories: [BugForge,daily,ottergram]
---

# Daily - Shady Oaks Finance
><br/><b>Vulnerabilities Covered:</b>
<br/>
IDOR (Insecure Direct Object Reference) via **web-sockets**
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates how an **`Insecure Direct Object Reference (IDOR)`** can surface in real-time features by moving beyond basic HTTP endpoints and testing **`WebSocket traffic`**. After creating an account and observing WebSocket activity used for messaging, the messaging workflow was analyzed by manipulating user-controlled identifiers such as recipient_id and later messageId within WebSocket events. By sending messages to oneself and inspecting notification traffic, it became clear that the backend trusted client-supplied message identifiers without validating ownership. Modifying these identifiers allowed access to other users’ messages, highlighting the importance of testing authorization controls not only on REST APIs but also on WebSockets and background real-time channels during penetration testing.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution
### Step 1 - Account Creation & Initial Review
Register a new user account and intercept the registration request and response. Review the payload for any role, privilege, or identifier fields assigned during account creation that could later influence authorization decisions.

![User registration request](/images/bug-forge/daily/ottergram/idor-web-sockets/registration-request-response.png)

---

### Step 2 - Observe WebSocket Activity After Login
Log into the application and monitor the network traffic. Identify active WebSocket connections, as these are commonly used for real-time functionality such as messaging and notifications.

![WebSocket calls](/images/bug-forge/daily/ottergram/idor-web-sockets/socket-requests.png)

---

### Step 3 - Analyze the Messaging Functionality
Send a message to the admin user and intercept the POST request to `/api/messages`. Inspect the request body and note the presence of the `recipient_id` parameter, which appears to control message delivery.

![Admin message](/images/bug-forge/daily/ottergram/idor-web-sockets/message-request.png)

Modify the `recipient_id` to your own user ID (`4`) and resend the request to test how the application handles self-referential messaging.

![Sending a message to yourself](/images/bug-forge/daily/ottergram/idor-web-sockets/self-messaging.png)

---

### Step 4 - Inspect WebSocket Message Handling
After sending a message to yourself, observe that a notification toaster appears in the top-right corner of the application.

![Toaster received](/images/bug-forge/daily/ottergram/idor-web-sockets/notification-toaster.png)

Open **Burp Suite → WebSockets History** and inspect the incoming WebSocket messages. Notice a `preview-message` event containing a `messageId` value.

![Preview message WebSocket](/images/bug-forge/daily/ottergram/idor-web-sockets/web-socket-preview-message.png)

To test for an Insecure Direct Object Reference (IDOR), modify the `messageId` value to reference other messages. This allows access to messages that do not belong to the current user, resulting in disclosure of unauthorized content.

![Flag](/images/bug-forge/daily/ottergram/idor-web-sockets/flag.png)
---

### Impact
- Unauthorized access to other users’ private messages  
- Exposure of sensitive or confidential communications  
- Loss of user trust in the platform  
- Demonstrates broken authorization controls in real-time functionality

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control  
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)  
- **Attack Surface:** WebSocket-based messaging  
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The backend trusts client-supplied identifiers such as `recipient_id` and `messageId` without validating that the authenticated user is authorized to access the referenced message objects. Authorization checks are missing or inconsistently applied between REST endpoints and WebSocket handlers.

---

### Remediation
- Enforce server-side authorization checks for all message operations, including WebSocket events  
- Validate message ownership before allowing read, preview, or delivery actions  
- Avoid using client-controlled identifiers as the sole basis for access decisions  
- Apply consistent access control logic across REST APIs and WebSocket implementations  
- Add logging and monitoring to detect unauthorized access attempts in real-time channels

---