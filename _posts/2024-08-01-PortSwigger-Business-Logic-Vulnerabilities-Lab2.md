---
layout: post
title:  "PortSwigger - Business Logic Vulnerabilities - Lab 2"
date:   2024-07-31 10:43
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [business-logic-vulnerability]
categories: [PortSwigger]
---

# Lab 2 - High-level logic vulnerability
><b>Lab Objective:</b>
<br/><br/>
This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket"
<br/><br/>
You can log in to your own account using the following credentials: `wiener:peter`
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-high-level">Port Swigger - Lab 2</a>
<br/>
<a href="https://academy.ranakhalil.com/courses/1491236/lectures/44961563">Rana Khalil - Business Logic Vulnerability -  Lab2</a>
<br/>
<a href="https://github.com/JacoZwarts/PortSwigger-Labs/blob/main/5.%20Business%20Logic%20Vulnerabilities/Labs/Lab%202%20-%20High-level%20logic%20vulnerability/Business-Logic-Vulnerability-Lab2.py">Python Script</a>
<br/>



### Solution
<hr/>

#### 1. Login in with provided credentials `wiener:peter`
#### 2. Click on `Home` to see the shop.

![Business Logic Vulnerability Shop](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-Store-Front.png)

#### 3. Click on view Details for the `Lightweight "l33t" Leather Jacket` product

![Business Logic Vulnerability - Product Detail](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-l33t-Leather-Jacket-Details.png)

#### 4. Add `Lightweight "l33t" Leather Jacket` to your cart.

#### 5. Add another item `Eggtastic, Fun, Food Eggcessories` to your cart and intercept the request

![Business Logic Vulnerability - Second Product](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-Second-Product.png)

![Business Logic Vulnerability - Second Product Details](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-Second-Product-Details.png)

![Business Logic Vulnerability - Second Product Request](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-Second-Product-Request.png)

#### 6. Change the quantity value to `-83`
`This will add a negative value of 15.94 * -83 = $1323.02 to our cart`

![Business Logic Vulnerability - Second Product Request](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-Intercepted-Request.png)

#### 7. Navigate to your cart and notice the total amount for the order is `$13.98`

![Business Logic Vulnerability - Second Product Request](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-l33t-Leather-Jacket-Order.png)

#### 8. Click on `Place Order` we've now purchased the `Lightweight "l33t" Leather Jacket` for `$13.98`

![Business Logic Vulnerability - Second Product Request](/images/portswigger/BusinessLogic/Lab2/Business-Logic-Lab2-Solved.png)

<hr/>