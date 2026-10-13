---
title: "INSERT-based SQL injection in MySQL"
order: 5
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

Error functions placed in an inserted value leak data even without reflection, because the error is raised while the row is built. If the statement is `INSERT INTO log (ip, note) VALUES ('<inj>', 'x')`, injecting a subquery value makes `EXTRACTVALUE` fail with the target string in the message:

```sql
1.2.3.4',(SELECT EXTRACTVALUE(1,CONCAT(0x7e,(SELECT password FROM users LIMIT 1)))))-- 
```

The more impactful primitive is `ON DUPLICATE KEY UPDATE`. It has no `WHERE` clause: the row it updates is decided entirely by which existing row the inserted values collide with on a `UNIQUE` or `PRIMARY` key. So to overwrite a specific account, inject that account's own unique value (here the admin email) so the insert collides with the admin row, and let the `UPDATE` branch rewrite the password. For `INSERT INTO users (email, password) VALUES ('<inj>', '<hash>')`:

```sql
admin@site.com','x') ON DUPLICATE KEY UPDATE password='$2y$10$attackerhash'-- 
```

The `email` value `admin@site.com` duplicates the existing admin row, so instead of adding an account the statement runs `UPDATE ... SET password='$2y$10$attackerhash'` on the admin row, a direct takeover. The exact payload depends on the real column list and which column carries the unique constraint, both recovered first with the enumeration techniques.

## Tools

- **sqlmap**: detects and exploits the injectable INSERT parameter, including error-based leaks.
- **Burp Repeater**: craft `ON DUPLICATE KEY UPDATE` and error-function payloads by hand.

## References

- MySQL Reference Manual: INSERT, INSERT ... ON DUPLICATE KEY UPDATE
- OWASP Testing Guide: Testing for SQL Injection
