---
layout: post
title:  "BugForge - Weekly - Furhire"
date:   2026-01-11 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   []
categories: [BugForge]
---

# Weekly - Furhire
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQLI Injection<br/>
JWT Signing<br/>
<br/>
<b>Summary:</b>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution


### Step 1 – User registration
Register a new user account and intercept the registration request and response. Review the payload for any role, privilege, or identifier fields assigned during account creation that could later influence authorization decisions.

---

### Step 2 – Fuzz directories

Found admin portal - Unauthorized error

---

### Step 3 – Job Get BY Id sqli
---

### Step 4 – JWT Signing

---

### Step 5 - Use new JWT

Use new jwt and retrieve flag from admin portal

---



