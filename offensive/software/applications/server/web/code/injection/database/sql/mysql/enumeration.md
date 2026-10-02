---
title: "Fingerprinting and enumeration in MySQL injection"
description: "Orienting a MySQL injection: confirming the engine, reading version and current context, and checking the account's privileges before choosing a technique."
keywords:
  - MySQL fingerprinting
  - version detection
  - current_user database
  - privilege enumeration
  - secure_file_priv
---

# Enumeration

Before picking a technique, confirm the engine is MySQL and read the context the injection runs in. The choice between union, error, file, and command routes depends on the version and on what the database account is allowed to do, so enumeration comes first.

Version and identity come from system functions and variables, read through whatever channel is open (a reflected column here):

```sql
' UNION SELECT CONCAT_WS(0x3a,@@version,@@version_comment,@@hostname),current_user(),database()-- 
```

`@@version` distinguishes MySQL from MariaDB (whose version string contains `MariaDB`) and gives the exact release, which decides version-gated behavior. `current_user()` is the account the server authenticated, `user()` is the login the client supplied, and `database()` is the current schema.

Privileges decide how far the injection goes. The account's grants are in `information_schema.user_privileges`, and the file-access setting is in `secure_file_priv`:

```sql
' UNION SELECT GROUP_CONCAT(privilege_type),NULL,NULL FROM information_schema.user_privileges WHERE grantee=CONCAT(0x27,REPLACE(current_user(),0x40,0x274027),0x27)-- 
' UNION SELECT @@secure_file_priv,NULL,NULL-- 
```

Seeing `FILE` in the privilege list, and a `secure_file_priv` that is empty rather than a restrictive path or `NULL`, is what tells you the file-read, out-of-band, and command-execution routes are reachable. A `SUPER` or admin privilege similarly signals that the UDF route and server-variable changes are possible. With the engine, version, and privileges known, the rest of the techniques apply with the right expectations.

## References

- MySQL Reference Manual: information functions, `information_schema.user_privileges`, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
