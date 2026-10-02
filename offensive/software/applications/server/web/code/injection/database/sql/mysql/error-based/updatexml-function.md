---
title: "UPDATEXML XPath errors for data exfiltration in MySQL error-based SQL injection"
description: Using UPDATEXML with malformed XPath to leak substrings through MySQL error messages.
keywords:
  - UPDATEXML
  - error-based SQLi
  - MySQL
---

# UPDATEXML

## Context

`UPDATEXML(xml_target, xpath, new_val)` **shares** **the** **same** **XPath** **error** **surface** **as** **`EXTRACTVALUE`** **for** **data** **exfiltration** **via** **errors**. **Payload** **patterns** **mirror** **`EXTRACTVALUE`** **with** **function** **name** **swap**.

## Theory

**Some** **WAFs** **signature** **one** **function** **but** **not** **the** **other**; **rotate** **both** **in** **testing**.

## Practice

```sql
AND UPDATEXML(1, CONCAT(0x7e,(SELECT user()),0x7e), 1)
```

**Escalate** **to** **table** **data** **with** **chunked** **`SUBSTRING`** **when** **errors** **truncate**.

## Tools

- **sqlmap**
- **Burp Suite**
