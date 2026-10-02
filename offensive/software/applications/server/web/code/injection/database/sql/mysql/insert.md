---
title: "INSERT-based SQL injection in MySQL"
description: "Exploiting injection inside a MySQL INSERT statement: leaking data through error functions in VALUES and overwriting rows with ON DUPLICATE KEY UPDATE."
keywords:
  - INSERT injection
  - ON DUPLICATE KEY UPDATE
  - account takeover
  - error based INSERT
  - MySQL write injection
---

# INSERT-based

Injection often lands in an `INSERT` rather than a `SELECT`, for example in a registration, comment, or logging form. The result is no reflected rows to read, but an `INSERT` still exposes two useful primitives.

Error functions placed in the inserted values leak data even without reflection, because the error is raised while the row is built:

```sql
x', (SELECT EXTRACTVALUE(1,CONCAT(0x7e,(SELECT password FROM users LIMIT 1)))))-- 
```

The more impactful primitive is `ON DUPLICATE KEY UPDATE`. If the target table has a unique column (such as `email` or `username`) and you can extend the `INSERT`, a deliberate key collision lets you update the existing row instead of adding one. Against a users table that means overwriting another account's fields:

```sql
attacker@site.com','hash') ON DUPLICATE KEY UPDATE password='attacker_hash' WHERE email='admin@site.com'-- 
```

Supplying the victim's unique value makes the insert collide with their row, and the `UPDATE` branch rewrites the password to one the attacker controls, a direct account takeover. The exact payload depends on the real column list and which value carries the unique constraint, both of which are recovered first with the enumeration techniques.

## References

- MySQL Reference Manual: INSERT, INSERT ... ON DUPLICATE KEY UPDATE
- OWASP Testing Guide: Testing for SQL Injection
