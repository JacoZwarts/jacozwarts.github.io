---
layout: post
title:  "PortSwigger - Business Logic Vulnerabilities - Lab 1"
date:   2024-07-31 10:43
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [business-logic-vulnerability]
categories: [PortSwigger]
---

# Lab 1 - Excessive trust in client-side controls
><b>Lab Objective:</b>
<br/><br/>
This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".
<br/><br/>
You can log in to your own account using the following credentials: `wiener:peter`
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls">Port Swigger - Lab 1</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44961556">Rana Khalil - Business Logic Vulnerability -  Lab1</a>
<br/>
<a href="https://github.com/JacoZwarts/PortSwigger-Labs/blob/main/5.%20Business%20Logic%20Vulnerabilities/Labs/Lab%201%20-%20Excessive%20trust%20in%20client-side%20controls/Business-Logic-Vulnerability-Lab1.py">Python Script</a>
<br/>



### Solution
<hr/>

#### 1. Login in with provided credentials `wiener:peter`
#### 2. Click on `Home` to see the shop.

![Business Logic Vulnerability Shop](/images/portswigger/BusinessLogic/Business-Logic-Lab1-Shop.png)

#### 3. Click on view Details for the `Lightweight "l33t" Leather Jacket` product

![Business Logic Vulnerability - Product Detail](/images/portswigger/BusinessLogic/Business-Logic-Lab1-Product_Detail.png)

#### 4. Intercept the request to Add `Lightweight "l33t" Leather Jacket` to your cart.

![Business Logic Vulnerability - Cart Request](/images/portswigger/BusinessLogic/Business-Logic-Lab1-Add-Cart_Request.png)

#### 5. Change the price in the above request
![Business Logic Vulnerability - Cart Request Price adjust](/images/portswigger/BusinessLogic/Business-Logic-Lab1-Cart-Request-Price.png)

#### 6. Navigate to your cart and purchase the `Lightweight "l33t" Leather Jacket` for $0.01
![Business Logic Vulnerability - Purchase item](/images/portswigger/BusinessLogic/Business-Logic-Lab1-Product-Purchase.png)
![Business Logic Vulnerability - Lab 1 Solved](/images/portswigger/BusinessLogic/Business-Logic-Lab1-Solved.png)

<hr/>