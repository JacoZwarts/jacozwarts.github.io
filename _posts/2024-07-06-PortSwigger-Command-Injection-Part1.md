---
layout: post
title:  "PortSwigger - Command injection Part 1"
date:   2024-07-06 13:40
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [BSCP,PortSwigger,Command Injection]
categories: [BSCP]
---

# Introduction
>OS Command Injection is a vulnerability that consists of an attacker executing commands on the host operating system via a vulnerable application.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/os-command-injection">PortSwigger - OS Command Injection</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/38308201">Rana Khalil Academy - OS Command injection</a>

### Two conditions for OS Command Injection vulnerabilities:
1. Uses a function that can execute system commands such as exec.
1. The parameter of executable methods are user controllable.

![Code vulnerable to command injection](/images/portswigger/CommandInjection/VulnerableCommandInjectionCodeExample.png)
