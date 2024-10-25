---
layout: post
title:  "PortSwigger - Authentication - Lab 14"
date:   2024-06-06 18:20
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [authentication,bscp]
categories: [PortSwigger]
---

# Lab 14 - 2FA bypass using a brute-force attack
><b>Lab Objective:</b>
<br/><br/>
This lab's two-factor authentication is vulnerable to brute-forcing. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, brute-force the 2FA code and access Carlos's account page.<br/>
`Victim's credentials: carlos:montoya`
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack">Port Swigger - Lab 14</a>
<br/>

### Solution
<hr/>

#### 1. With Burp running, log in as `carlos` and investigate the 2FA verification process. Notice that if you enter the wrong code twice, you will be logged out again. You need to use Burp's session handling features to log back in automatically before sending each request.

#### 2. In Burp, go to `Project options > Sessions`. In the `Session Handling Rules` panel, click `Add`. The `Session handling rule` editor dialog opens.

#### 3. In the dialog, go to the `Scope` tab. Under `URL Scope`, select the option `Include all URLs`.

#### 4. Go back to the `Details` tab and under `Rule Actions`, click `Add > Run a macro`.

#### 5. Under `Select macro` click `Add` to open the `Macro Recorder`. Select the following 3 requests: `GET /login POST /login GET /login2` Then click `OK`. The `Macro Editor` dialog opens.

#### 6. Click `Test macro` and check that the final response contains the page asking you to provide the 4-digit security code. This confirms that the macro is working correctly.

#### 7. Keep clicking `OK` to close the various dialogs until you get back to the main Burp window. The macro will now automatically log you back in as Carlos before each request is sent by Burp Intruder.

#### 8. Send the `POST /login2` request to Burp Intruder

#### 9. In Burp Intruder, add a payload position to the `mfa-code` parameter

#### 10. On the `Payloads` tab, select the Numbers payload type. Enter the range 0 - 9999 and set the step to 1. Set the min/max integer digits to 4 and max fraction digits to 0. This will create a payload for every possible 4-digit integer.

#### 11. Go to the `Resource pool` tab and add the attack to a resource pool with the `Maximum concurrent requests` set to `1`.

#### 12. Start the attack. Eventually, one of the requests will return a `302` status code. Right-click on this request and select `Show response in browser`. Copy the URL and load it in the browser.

#### 13. Click `My account` to solve the lab.
<hr/>

### Explanation:
<hr/>
When logging in with the credentials we're prompted to supply a 2FA code. When trying to brute force the 2FA code we get logged out after two attempts, we'll use BurpSuite's macro functionality to automatically log us back in and then continue brute forcing the 2FA code.
<hr/>

### Steps to Configure a Macro in Burp:
<hr/>

#### 1. Navigate to Settings at the top righthand corner:
![Configure Macro Step 1](/images/portswigger/Authentication/configure-macro-step1.png)
#### 2. Click on the Sessions Tab:
![Configure Macro Step 2](/images/portswigger/Authentication/configure-macro-step2.png)
#### 3. In the Session handling rules click on the Add button
![Configure Macro Step 3](/images/portswigger/Authentication/configure-macro-step3.png)
#### 4. Click on the Scopes Tab:
![Configure Macro Step 4](/images/portswigger/Authentication/configure-macro-step4.png)
#### 5. In the URL scope section select Include all URLs:
![Configure Macro Step 5](/images/portswigger/Authentication/configure-macro-step5.png)
#### 6. Click on the Details Tab:
![Configure Macro Step 6](/images/portswigger/Authentication/configure-macro-step6.png)
#### 7. In the Rule actions section click on Add:
![Configure Macro Step 7](/images/portswigger/Authentication/configure-macro-step7.png)
#### 8. Select Run a macro:
![Configure Macro Step 8](/images/portswigger/Authentication/configure-macro-step8.png)
#### 9. Click on the Add button:
![Configure Macro Step 9](/images/portswigger/Authentication/configure-macro-step9.png)
#### 10. Select the request you want to execute in the Macro and click on OK.
![Configure Macro Step 10](/images/portswigger/Authentication/configure-macro-step10.png)
#### 11. Test the macro by clicking on Test Macro
![Configure Macro Step 11](/images/portswigger/Authentication/configure-macro-step11.png)
#### 12.When using intruder set resource pool to send one request at a time:
![Configure Macro Step 12](/images/portswigger/Authentication/configure-macro-step12.png)
<hr/>