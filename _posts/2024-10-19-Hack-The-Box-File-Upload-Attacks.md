---
layout: post
title:  "Hack The Box - Academy - File Upload Attacks"
description: "Follow this comprehensive walkthrough of the Hack The Box Academy File Upload Attacks Skills Assessment. Master file upload attack techniques to exploit vulnerable web applications with expert guidance from Zwarts Sec, perfect for enhancing your penetration testing skills and preparing for HTB challenges."
date:   2024-10-19 14:24
image:  /images/htb/file-upload-attacks/logo.png
tags:   [file-upload-attacks,cbbh]
categories: [htbacademy]
---

>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/136/section/1259">File Upload Attacks</a><br/>
<a href="https://jacozwarts.github.io/images/htb/file-upload-attacks/File_Upload_Attacks_Module_Cheat_Sheet.pdf">File Upload Attacks - Cheat Sheet</a><br/>
<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst">PayloadsAllTheThings - PHP Extensions List</a><br/>

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

![Caido White list Request](/images/htb/file-upload-attacks/white-list-filter-caido-request.png)

![Caido White list Result](/images/htb/file-upload-attacks/white-list-filter-caido-result.png)

![Webshell](/images/htb/file-upload-attacks/white-list-filter-web-shell.png)

Navigate through the folders to locate the `flag.txt` file.

![Webshell](/images/htb/file-upload-attacks/white-list-filter-flag-result.png)
<hr/>