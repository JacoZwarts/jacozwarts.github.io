---
layout: post
title:  "Hack The Box - Academy - SQLMap Essentials"
date:   2024-10-11 12:33
image:  /images/htb/sqlmap_essentials/logo.png
tags:   [sqlmap,cbbh]
categories: [htbacademy]
---

><b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/58/section/509">SQLMap Essentials</a><br/>
<a href="https://jacozwarts.github.io/images/htb/sqlmap_essentials/Sqlmap_Essentials_Module_Cheat_Sheet.pdf">SQLMap - Cheat Sheet</a>
<br/>

### Intercepting and Saving Requests via Burp Suite:
1. `Enable FoxyProxy`: Ensure that FoxyProxy is enabled and correctly configured to route traffic through Burp Suite.
1. `Capture the Request`: In Burp Suite, intercept the HTTP request you want to analyze.
1. `Save the Request`: Right-click on the intercepted request and select Copy to File. Choose a location on your system to save the request.
![Save Burp Suite Request](/images/htb/sqlmap_essentials/burp-suite-save-request.png)
<hr/>

### Running SQLMap on an HTTP Request - Exercises

#### What's the contents of the table flag2 (Case #2) - [Post parameter]
```
sqlmap -r Case2.txt --threads 10 --dump -T flag2 --batch
```

- `-r Case2.txt`: This option specifies that SQLmap should read the HTTP request from the file Case2.txt. The file typically contains the raw HTTP request, including headers, cookies, etc., needed for the injection test.
- `--threads 10`: Runs SQLmap using 10 threads to speed up the scanning process.
- `--dump -T flag2`: Instructs SQLmap to dump (extract) all data from the table named flag2.
- `--batch`: Runs SQLmap in non-interactive mode, automatically using default options when prompts appear, which is useful for scripting or automation.

#### What's the contents of table flag3? (Case #3) - [Cookie value]
Save the HTTP request intercepted by Burp Suite, and observe that the `id` parameter is passed through the `Cookie` header.
```
sqlmap -r Case3.txt -p cookie --threads 10 --dump -T flag3 --batch
```
- `-p cookie:` This flag specifies the parameter to be tested for SQL injection. In this case, it’s instructing sqlmap to focus on the cookie parameter found in the request. If the cookie contains something like id=123, SQLmap will target that part for testing.

#### What's the contents of table flag4? (Case #4) - [JSON value]
```
sqlmap -r Case4.txt --threads 10 --dump -T flag4 --batch
```
<hr/>

### Attack Tuning - Exercises

#### What's the contents of table flag5? (Case #5) - [OR SQLi]

`The Hack The Box machine is lagging, resulting in incorrect decoding. For example: HTB{700_mucl_yIIyaqu7_w0r7h_17}. To ensure accurate decoding, use the --time-sec option to mitigate issues caused by the lagging server.`

![HTB Lagging](/images/htb/sqlmap_essentials/case5-lagging.png)

```
sqlmap -r Case5.txt --batch --dump -T flag5 -D testdb --no-cast --dbms=MySQL --technique=T --time-sec=10 --level=5 --risk
=3 --fresh-queries
```
- `--no-cast`: This option disables automatic type casting of data. When enabled, `sqlmap` tries to cast data types (e.g., converting strings to integers), which might not be necessary or could lead to incorrect results. Disabling it can sometimes provide more accurate outcomes.
- `--dbms=MySQL`: This flag specifies the database management system to target. By setting it to `MySQL`, you instruct `sqlmap` to use techniques specifically for exploiting MySQL databases.
- `--technique=T`: This option tells sqlmap to use only the "time-based blind" SQL injection technique for testing. The "T" represents time-based blind SQL injection, which relies on the time it takes to receive a response to infer information about the database.
- `--time-sec=10`: Sets the number of seconds to wait before considering a response to be delayed. In this case, sqlmap will wait for 10 seconds to determine if the server is responding slowly, which helps in time-based SQL injection techniques.
- `--level=5`: This option sets the level of tests to perform. Levels range from 1 to 5, with level 5 being the most thorough. It increases the number of tests that sqlmap conducts, which can lead to a better chance of discovering vulnerabilities.
- `--risk=3`: Sets the risk level for tests. Risk levels range from 0 to 3, with 3 being the highest. Higher risk levels enable more aggressive tests, which may be more likely to exploit vulnerabilities.
- `--fresh-queries`: This flag tells sqlmap to ignore cached data and to re-execute all queries against the database. This ensures that any changes to the database since the last query execution are accounted for.

#### What's the contents of table flag6? (Case #6) - [Non-standard boundaries]

```
sqlmap -r Case6.txt --batch --dump -T flag6 -D testdb --no-cast --level=5 --risk=3 --prefix='`)'
```
- `--prefix=`: This flag allows you to specify a prefix to prepend to the extracted data.

#### What's the contents of table flag7? (Case #7) - [UNION SQLi with adjustments]
`We should verify that the number of columns in the UI is 5`
```
sqlmap -r Case7.txt --batch --dump -T flag7 -D testdb --no-cast --level=5 --risk=3 --union-cols=5
```
- `--union-cols=5`: Specifies the number of columns to be used in a UNION query during the SQL injection exploitation. By setting this flag to 5, you are instructing sqlmap to assume that the vulnerable query returns 5 columns. This is important for constructing valid UNION queries, as both the original and injected queries must have the same number of columns to work correctly.

<hr/>

### Database Enumeration & Advanced Database Enumeration - Exercises
#### What's the contents of table flag1 in the testdb database? (Case #1)
```
```

#### What's the name of the column containing "style" in it's name? (Case #1)
```
```

#### What's the Kimberly user's password? (Case #1)
```
```

<hr/>