---
title: "EXTRACTVALUE XPath errors for data exfiltration in MySQL error-based SQL injection"
description: Using EXTRACTVALUE with invalid XPath to surface query substrings inside MySQL error text.
keywords:
  - EXTRACTVALUE
  - error-based SQLi
  - MySQL
---

# EXTRACTVALUE

## Context

`EXTRACTVALUE(xml_frag, xpath)` **raises** **XPATH** **syntax** **errors** **that** **embed** **the** **evaluated** **fragment** **when** **xpath** **is** **attacker**-**controlled** **or** **when** **concat** **of** **secrets** **is** **passed** **into** **xpath**. **Classic** **payload** **shape** **uses** **`concat(0x7e,(SELECT ...),0x7e)`** **inside** **a** **path** **that** **fails** **parsing** **and** **reflects** **data** **in** **the** **error** **string**.

## Theory

**Works** **best** **when** **stacked** **queries** **or** **expression** **contexts** **allow** **functions**; **disabled** **on** **some** **hardened** **MySQL** **configs**. **Length** **limits** **truncate** **error** **messages**—**use** **`SUBSTRING`** **in** **chunks**.

## Practice

### Single-row leak in lab

```sql
AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT password FROM users LIMIT 1), 0x7e))
```

Adapt **quotes** **and** **parentheses** **to** **the** **injection** **point**.

## Tools

- **Burp Suite**
- **sqlmap**
