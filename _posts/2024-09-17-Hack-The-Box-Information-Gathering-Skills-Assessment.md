---
layout: post
title:  "Hack The Box - Academy - Information Gathering Skills Assessment"
description: "Master information gathering techniques in Hack The Box - Academy - Information Gathering Skills Assessment. Prepare for the CBBH certificate with expert tips and strategies from Zwarts Sec to boost your cybersecurity knowledge."
date:   2024-09-17 21:05
image:  /images/htb/information-gathering/information-gathering.png
tags:   [recon,cbbh]
categories: [htbacademy]
---

### Explore this detailed walkthrough of Hack The Box Academy's Information Gathering Skills Assessment module.

><b>Overview:</b>
<br/><br/>
To complete the skills assessment, answer the questions below. You will need to apply a variety of skills learned in this module, including:<br/><br/>
Using whois<br/>
Analysing robots.txt<br/>
Performing subdomain bruteforcing<br/>
Crawling and analysing results<br/><br/>
Demonstrate your proficiency by effectively utilizing these techniques. Remember to add subdomains to your hosts file as you discover them.
<br/>
<br/>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/144/section/1311">Information Gathering - Web Edition</a><br/>
<a href="https://jacozwarts.github.io/images/htb/information-gathering/Information_Gathering_Web_Edition_Module_Cheat_Sheet.pdf">Information Gathering - Cheat Sheet</a>
<br/>

## Q1: What is the IANA ID of the registrar of the inlanefreight.com domain?

#### Execute the bellow command:

```
whois inlanefreight.com
```

![whois inlanefreight.com result](/images/htb/information-gathering/htb-academy-information-gathering-whois.png)

<hr/>

## Q2: What http server software is powering the inlanefreight.htb site on the target system? Respond with the name of the software, not the version, e.g., Apache.

#### 1. Execute the bellow command to add the Target_IP and hostname to /etc/hosts

```
sudo sh -c 'echo "{TARGET_IP} inlanefreight.htb" >> /etc/hosts'
```
#### 2. Execute the bellow command to retrieve the server banner, revealing the web server software and version number:

```
curl -I http://inlanefreight.htb:{PORT}
```
![Server Software nginx](/images/htb/information-gathering/server-software.png)

<hr/>

## Q3: What is the API key in the hidden admin directory that you have discovered on the target system?

#### 1. Execute the below command to find sub domains for inlanefreight.htb

```
gobuster vhost -u http://inlanefreight.htb:{PORT} -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```
![Sub domain found: web1337 Subdomain](/images/htb/information-gathering/web1337-subdomain.png)

#### 2. Execute the below command to add web1337 to /etc/hosts
```
sudo sh -c 'echo "{TARGET_IP} web1337.inlanefreight.htb" >> /etc/hosts'
```

#### 3. Navigate to the robots.txt `http://web1337.inlanefreight.htb:{PORT}/robots.txt` file and look for the Disallow directory:
![robots.txt content](/images/htb/information-gathering/robots-content.png)

#### 4. Execute the below command to enumerate directories / files
```
ffuf -u http://web1337.inlanefreight.htb:39535/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt:FUZZ -e .html -v
```
![API Key in Hidden admin directory](/images/htb/information-gathering/api-key-hidden-directory.png)
<hr/>

## Q4: After crawling the inlanefreight.htb domain on the target system, what is the email address you have found? Respond with the full email, e.g., mail@inlanefreight.htb. & Q5: What is the API key the inlanefreight.htb developers will be changing too?

#### 1. Execute the below command to find sub domains for web1337.inlanefreight.htb

```
gobuster vhost -u http://inlanefreight.htb:{PORT} -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```
![Sub domain found: dev Subdomain](/images/htb/information-gathering/dev.web1337-subdomain.png)

#### 2. Execute the below commands to install Scrapy and ReconSpider
```
pip3 install scrapy
```
```
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
```
```
unzip ReconSpider.zip
```
```
python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:{PORT}
```

### 3. Review the results
```
cat results.json
```
![ReconSpider Result](/images/htb/information-gathering/ReconSpider-Result.png)

<hr/>