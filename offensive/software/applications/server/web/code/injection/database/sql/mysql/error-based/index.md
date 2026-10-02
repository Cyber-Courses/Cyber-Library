---
title: "Error-based SQL injection (MySQL): EXTRACTVALUE, UPDATEXML, and duplicate-key channels"
description: Leaking data through MySQL error messages using UPDATEXML, EXTRACTVALUE, and GROUP BY duplicate-key errors.
keywords:
  - error-based SQLi
  - UPDATEXML
  - EXTRACTVALUE
  - MySQL
---

# Error-based SQLi

**Error-based** SQLi **forces** **functions** **that** **take** **XPath** **or** **malformed** **XML** **arguments** **to** **raise** **errors** **whose** **message** **includes** **evaluated** **substrings** **from** **secrets**. **GROUP** **BY** **duplicate** **entry** **errors** **can** **also** **leak** **concatenated** **data** **on** **older** **MySQL** **builds**.

## Pages

- [EXTRACTVALUE function](extractvalue-function.md)
- [UPDATEXML function](updatexml-function.md)
- [GROUP BY duplicate entry](group-by.md)
