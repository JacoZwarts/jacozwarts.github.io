# Weekly - FurHire
><b>Objective:</b>
<br/>
FurHire recently made some optimisation changes to their site. They are particularly interested in issues that could compromise user accounts and PII.
<br/>
<br/>
<b>Vulnerabilities Covered:</b>
<br/>
waf by pass<br/>
xss
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<a href="https://www.youtube.com/watch?v=0OMmWtU2Y_g">NahamCon 2024 – Modern WAF Bypass Techniques on Large Attack Surfaces</a>
<br/>
<a href="https://github.com/assetnote/nowafpls">Github: nowafpls</a>
<br/>

## Solution


### Step 1 – Account Creation
Create two user accounts:
- One account with the **JobSeeker** role
- One account with the **Recruiter** role

![Job Seeker - Sign up](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/Signup-job-seeker.png)


### Step 2 – Job Posting
Log in as the **Recruiter** and create a new job posting.

### Step 3 – Job Application
Log in as the **JobSeeker** and apply to the newly created job posting.

![Job Seeker - Application](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/Zwarts-Application.png)

![Job Seeker - Apply](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/Apply.png)



### Step 4 – Application Review
Switch back to the **Recruiter** account.
Review incoming applications and accept the JobSeeker’s application.

![Recruiter - Application Review](/images/bug-forge/weekly/fur-hire/waf-xss/Recruiter/Application-from-test.png)

### Step 5 – Notification Confirmation
Log back in as the **JobSeeker**.
Observe the notification indicating that the application has been accepted.

![Job Seeker - Status Update](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/Application-Accepted-Notification.png)


### Step 6 – Payload Analysis
Inspect the network traffic related to the application status update.
Identify and analyze the request payload responsible for updating the application state.


![Application Status Update - Request](/images/bug-forge/weekly/fur-hire/waf-xss/Recruiter/request-accept-application.png)

### Step 7 – Initial XSS Attempt
Attempt to inject a basic XSS payload into the request.
Note that the request is blocked, indicating the presence of a Web Application Firewall (WAF).

![Application Status Update - XSS Attempt](/images/bug-forge/weekly/fur-hire/waf-xss/Recruiter/xss-request-blocked-by-waf.png)

### Step 8 – WAF Bypass Research
Determine that a WAF bypass technique is required.
Reference 
[**NahamCon 2024 – Modern WAF Bypass Techniques on Large Attack Surfaces**](https://www.youtube.com/watch?v=0OMmWtU2Y_g)
 by Shubs for payload inflation techniques.

[**Github: nowafpls**](https://github.com/assetnote/nowafpls)


### Step 9 – Payload Inflation
Modify the request by adding an approximately **8KB payload** to evade WAF detection.

![Waf Bypass](/images/bug-forge/weekly/fur-hire/waf-xss/Recruiter/request-by-pass-waf.png)

Use below payload because `<script>` is removed

```
<img src=x onerror=alert(1)>
```

### Step 10 – XSS Execution
Return to the **JobSeeker** dashboard.
Confirm that the injected XSS payload is executed.

![XSS Trigger](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/XSS-Triggered.png)

### Step 11 – Additional Functionality Review
Review other application functionality.<br/>
Identify a password update feature that:
- Lacks CSRF protection
- Does not require the user’s current password

![Change Password UI](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/password-change-ui.png)
![Change Password Request](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/Password-change-request.png)


### Step 12 – XSS to Password Change
Craft a JavaScript payload that abuses the stored XSS to trigger a password update request.

Reference the application’s JavaScript code and the `apiRequest` helper function.
Base64-encode the malicious request body to avoid issues with special characters.

![apiRequest Helper](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/apiRequest-helper.png)
![btoa Payload](/images/bug-forge/weekly/fur-hire/waf-xss/Job%20Seeker/btoa-payload.png)


### Step 13 – Test Account Takeover
Execute the payload.
Confirm that the password for the test account has been updated successfully.

![Malicious-Payload](/images/bug-forge/weekly/fur-hire/waf-xss/Recruiter/malicious-payload-to-update-password.png)

### Step 14 – Flag Retrieval
Log in using the following credentials:
- **Username:** Jeremy
- **Password:** Test1234

Upon successful login, observe and capture the flag.

![Jeremy Account login](/images/bug-forge/weekly/fur-hire/waf-xss/Recruiter/jeremy-login-flag.png)

---