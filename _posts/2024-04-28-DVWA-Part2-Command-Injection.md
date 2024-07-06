---
layout: post
title:  "Damn Vulnerable Web Application - Part 2 / 12 Command Injection"
date:   2024-04-28 10:00
image:  /images/dvwa-walkthrough/DVWA.jpg
tags:   [appsec,php,owasp,brute-force,Command Injection]
categories: [Walkthrough]
---

# Command injection
>Command injection is an attack in which the goal is execution of arbitrary commands on the host operating system via a vulnerable application. Command injection attacks are possible when an application passes unsafe user supplied data (forms, cookies, HTTP headers etc.) to a system shell. In this attack, the attacker-supplied operating system commands are usually executed with the privileges of the vulnerable application. Command injection attacks are possible largely due to insufficient input validation. <br/><br/> This attack differs from Code Injection, in that code injection allows the attacker to add their own code that is then executed by the application. In Command Injection, the attacker extends the default functionality of the application, which execute system commands, without the necessity of injecting code.
<br/>
<br/>
<b>Reference:</b>
<a href="https://owasp.org/www-community/attacks/Command_Injection">OWASP</a>

### Security Level: `Low`

![DVWA Command Injection Initial page](/images//dvwa-walkthrough/command-injection/command-injection-low-initial-page.png)

1. We'll try the basic command injection payloads:
<br/><br/>
`; whoami #` <br/>
`& whoami #` <br/>
`| whoami #` <br/><br/>

![DVWA Command Injection Result Low](/images//dvwa-walkthrough/command-injection/command-injection-low-result.png)

#### Source Code:

![DVWA Command Injection Source Code Low](/images//dvwa-walkthrough/command-injection/command-injection-low-source-code.png)
<br/>

### Security Level: `Medium`

Use the basic `; whoami #` payload didn't work the `;` must be filtered out. Try with `& whoami #` works

#### Source Code:

![DVWA Command Injection Source Code Medium](/images//dvwa-walkthrough/command-injection/command-injection-medium-source-code.png)
<br/>

Looking at the source code we see `&&` and `;` is replaced with an empty string.

### Security Level: `High`

Iterating through the basic payloads we notice that none of the payloads gives us the result, when trying `|whoami #` we get a result. Looking at the source code below we can see that there is a typo in the filter for `|` and we can bypass this by removing the space between the `|` and `whoami #`.

#### Source Code:

![DVWA Command Injection Source Code High](/images//dvwa-walkthrough/command-injection/command-injection-high-source-code.png)
<br/>
