---
title: "Authentication bypass through MySQL injection"
description: "Bypassing a login whose SQL is built from the username and password fields, using comment termination and always-true conditions in MySQL."
keywords:
  - authentication bypass
  - login bypass
  - OR 1=1
  - comment injection
  - MySQL login
---

# Authentication bypass

A login that builds its query from the submitted fields, such as `SELECT * FROM users WHERE username='$u' AND password='$p'`, treats any returned row as a valid login. Injection into either field can make the query return the target row without the right password.

Commenting out the password check is the cleanest bypass. Supplying `admin'-- ` as the username closes the username string and comments the rest of the line:

```
username: admin'-- 
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

The `AND password` test is now inside a comment, so the query returns the admin row and the application logs in as admin. In MySQL the `-- ` comment needs a trailing space; `#` is an alternative that does not, and `admin'#` works where a trailing space is stripped.

An always-true condition works when no specific user is targeted. `' OR '1'='1` makes the `WHERE` match every row, and `LIMIT 1` keeps it to the first (often the first-created administrator):

```
username: ' OR '1'='1' LIMIT 1-- 
query:    SELECT * FROM users WHERE username='' OR '1'='1' LIMIT 1-- ' AND password='...'
```

The same payloads go in the password field when the username is fixed. The bypass depends only on the row being returned, so it works regardless of how the password is later checked, as long as the application equates a non-empty result set with a successful login.

## References

- MySQL Reference Manual: comment syntax, WHERE, LIMIT
- OWASP Testing Guide: Testing for SQL Injection
