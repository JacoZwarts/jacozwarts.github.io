---
layout: post
title:  "BugForge - Weekly - Galxy Dash"
date:   2026-01-19 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw,type-confusion]
categories: [BugForge,weekly,galaxy-dash]
---

# Weekly - Galxy Dash
><br/><b>Vulnerabilities Covered:</b>
<br/>
Business Logic Flaw
<br/>
Type confusion
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough highlights a multi-tenant business logic flaw where type confusion in a team management API allows tenant isolation to be bypassed. After ruling out cross-organization abuse in other features, testing revealed that changing the organization_id parameter to an unexpected data type bypassed backend validation, enabling user creation under a different organization. This demonstrates how weak schema enforcement and type assumptions can undermine authorization controls in multi-tenant systems.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Account Creation & Baseline Review

![Registration Request](/images/bug-forge/weekly/galaxy-dash/business-logic-flaw/type-confusion/registration-request.png)

The initial step was to review the account registration flow and establish a baseline understanding of how new users are created.  
The primary focus here was identifying any role, permission, or privilege-related fields that are assigned or implicitly trusted during account creation.

---

### Step 2 - Delivery Functionality Analysis

The goal of this phase was to determine whether it was possible to:
- Reduce the cost of deliveries, or  
- Create deliveries on behalf of a different organization

A significant amount of time was spent testing the delivery functionality. During this process, several issues were identified:
- The application could be forced into a broken state, causing the React UI to return a `500` error  
- Deliveries could be created with invalid origin and destination locations  
  - These deliveries were successfully stored server-side but could not be viewed in the UI  
- A delivery could be cancelled by directly calling the `DELETE` endpoint while its status was still `PENDING`

Although these behaviors indicated multiple logic flaws, none directly enabled cross-organization abuse. At this point, testing shifted to another area of the application.

---

### Step 3 - Team Functionality Analysis

This phase focused on the team management feature, specifically testing whether a user could be assigned to a different organization in a multi-tenant environment.

The first step was to capture and analyze the **Add Team Member** request.

![Team Management UI](/images/bug-forge/weekly/galaxy-dash/business-logic-flaw/type-confusion/team-management-ui.png)

The following endpoints were identified as part of the team management functionality:

- `GET` `/api/teams`  
- `POST` `/api/teams`  
- `PUT` `/api/teams/{user_id}`  
- `DELETE` `/api/teams/{user_id}`  

A new team member was created as part of normal application behavior. During this process, the `organizationId` was returned in the server response.

![Add Team Member Request](/images/bug-forge/weekly/galaxy-dash/business-logic-flaw/type-confusion/post-team-request.png)

The original request was then modified to include a different `organization_id`, attempting to create a user for another organization. This attempt was initially blocked by backend validation.

![Add Team Member Request - Blocked](/images/bug-forge/weekly/galaxy-dash/business-logic-flaw/type-confusion/add-team-member-different-organization-blocked.png)

By altering the `organization_id` from its expected (integer) data type to a `string`, the backend validation logic was bypassed, allowing the user to be created under a different organization. The same behavior can also be achieved by changing the property type to an `array`, such as [1].

![Add Team Member Request - Bypassed](/images/bug-forge/weekly/galaxy-dash/business-logic-flaw/type-confusion/add-team-member-different-organization.png)

After logging in with the newly created user and navigating to the **Team** section, the challenge flag was visible, confirming successful exploitation.

![Flag](/images/bug-forge/weekly/galaxy-dash/business-logic-flaw/type-confusion/flag.png)

---

### Impact
- Unauthorized creation of users under a different organization in a multi-tenant application  
- Cross-tenant access that breaks organizational isolation  
- Potential exposure of sensitive team, delivery, or organizational data  
- Privilege and trust boundary violations within the application  
- Demonstrates a business logic flaw that can be exploited to pivot across tenants  

---

### Vulnerability Classification
- **OWASP Top 10:** Broken Access Control  
- **Vulnerability Type:** Business Logic Flaw / Type Confusion  
- **Attack Surface:** Team management and user provisioning API  
- **CWE:** CWE-843 - Access of Resource Using Incompatible Type  

---

### Root Cause
The backend relies on insufficient input validation and inconsistent type handling for the `organization_id` field. While authorization checks exist for correctly typed values, changing the data type bypasses validation logic. This allows client-controlled manipulation of tenant identifiers, resulting in users being created under unauthorized organizations without proper server-side enforcement.

---

### Remediation
- Enforce strict server-side schema validation, including data type enforcement for all tenant identifiers  
- Reject requests containing unexpected or mismatched data types at the API boundary  
- Validate organizational ownership and authorization independently of client-provided identifiers  
- Centralize tenant resolution logic to avoid fragmented or inconsistent validation paths  
- Add security tests and monitoring for multi-tenant boundary violations and anomalous user creation patterns  

---