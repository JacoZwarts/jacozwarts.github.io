---
layout: post
title:  "BugForge - Daily - Shady Oaks Finance (Repeat)"
date:   2026-01-30 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,parameter-tampering,insecure-design]
categories: [BugForge]
---

# Daily - Shady Oaks Finance (Repeat)

This is a repeat of the daily challenge from January 2nd, 2026.

---

## Vulnerability Overview

This challenge demonstrates a **broken access control vulnerability caused by insecure design**. The application trusts client-supplied input to set sensitive user attributes, specifically the `role` parameter during an account upgrade process.

**Key Issues:**
- The application accepts a `role` parameter directly from the client during the upgrade request
- No server-side validation or authorization checks are performed on the role value
- An attacker can intercept the upgrade request and change the role to `administrator`
- The backend blindly accepts the tampered value, granting administrative privileges

This represents a fundamental design flaw where authorization decisions are delegated to the client rather than being enforced server-side.

---

## Vulnerabilities Covered

- **Broken Access Control** - Unauthorized privilege escalation through parameter manipulation
- **Parameter Tampering** - Modifying the `role` parameter in the upgrade request
- **Insecure Design** - Trusting client input for security-critical role assignment

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

## Full Solution

For the complete step-by-step walkthrough, see the original post:

[BugForge - Daily - Shady Oaks Finance (January 2nd, 2026)](https://zwarts.dev/posts/2026/01/02/BugForge-Daily-Shady-Oaks-Finance/)

---
