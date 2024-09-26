---
layout: post
title:  "Hack The Box - Academy - Javascript Deobfuscation Skills Assessment"
date:   2024-09-26 19:43
image:  /images/htb/js-deobfuscation/js-deobfuscation-logo.png
tags:   [js-deobfuscation]
categories: [htbacademy]
---

><b>Overview:</b>
<br/><br/>
You are given an online academy's IP address but have no further information about their website. As the first step of conducting a Penetration Test, you are expected to locate all pages and domains linked to their IP to enumerate the IP and domains properly.<br/><br/>
Finally, you should do some fuzzing on pages you identify to see if any of them has any parameters that can be interacted with. If you do find active parameters, see if you can retrieve any data from them.
<br/>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/41/section/519">Javascript Deobfuscation</a>
<br/>

## Q1: Try to study the HTML code of the webpage, and identify used JavaScript code within it. What is the name of the JavaScript file being used?

#### Navigate to the {Target} you want to inspect using your preferred browser. Open the `Developer Tools` (usually accessible via F12 or right-click > Inspect) and go to the `Sources tab`.

![Q1 Javascript code file](/images/htb/js-deobfuscation/Q1-source-javascript.png)
### Answer: 
`api.min.js`
<hr/>

## Q2: Once you find the JavaScript code, try to run it to see if it does any interesting functions. Did you get something in return?
Notice the log in the `Console` tab 
![Q2 Console log](/images/htb/js-deobfuscation/Q2-console-log.png)
### Answer: 
`HTB{j4v45cr1p7_3num3r4710n_15_k3y}`
<hr/>

## Q3: As you may have noticed, the JavaScript code is obfuscated. Try applying the skills you learned in this module to deobfuscate the code, and retrieve the 'flag' variable.


#### Use <a href="https://matthewfl.com/unPacker.html">Unpacker</a> to unpack the content in `api.min.js`
![Q3 Deobfuscated javascript](/images/htb/js-deobfuscation/Q3-deobfuscate-js-file.png)

#### Notice the flag variable in the code:
![Q3 Flag value](/images/htb/js-deobfuscation/Q3-flag.png)


### Answer: 
`HTB{n3v3r_run_0bfu5c473d_c0d3!}`
<hr/>

## Q4: Try to Analyze the deobfuscated JavaScript code, and understand its main functionality. Once you do, try to replicate what it's doing to get a secret key. What is the key?

#### After investigating the code, notice it's making an HTTP Post request to `{Target}/keys.php`

![Q4 Javascript Logic](/images/htb/js-deobfuscation/Q4-Logic.png)


#### Use the following curl command to make a POST request to `{Target}/keys.php`

```
curl -X POST {Target}/keys.php
```

![Q4 POST Result](/images/htb/js-deobfuscation/Q4-Initial-Post.png)


### Answer: 
`4150495f70336e5f37333537316e365f31355f66756e`
<hr/>

## Q5:  Once you have the secret key, try to decide it's encoding method, and decode it. Then send a 'POST' request to the same previous page with the decoded key as "key=DECODED_KEY". What is the flag you got?

#### Execute the following command to decrypt the payload received from question 4

```
echo 4150495f70336e5f37333537316e365f31355f66756e |xxd -p -r
```

#### Execute the following command to retrieve the flag

```
curl -X POST http://{Target}/keys.php -d "key=API_p3n_73571n6_15_fun"
```
![Q5 POST Result](/images/htb/js-deobfuscation/Q5-Result.png)

### Answer: 
`HTB{r34dy_70_h4ck_my_w4y_1n_2_HTB}`
<hr/>

