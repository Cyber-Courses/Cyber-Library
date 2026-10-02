---
title: "Authentication bypass through PostgreSQL injection"
description: "Bypassing a login whose SQL is built from the username and password fields in PostgreSQL, using comment termination and always-true conditions."
keywords:
  - authentication bypass
  - login bypass
  - OR 1=1
  - comment injection
  - PostgreSQL login
---

# Authentication bypass

A login that interpolates the submitted fields, such as `SELECT * FROM users WHERE username='$u' AND password='$p'`, authenticates on any returned row. Injecting into either field can make it return the target row without the correct password.

Commenting out the password check logs in as a chosen user. In PostgreSQL the `--` comment does not need a trailing space, so `admin'--` closes the string and removes the rest of the line:

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

The password test is commented out, so the admin row is returned and the application treats the login as successful. An always-true condition returns the first row when no specific account is targeted:

```
username: ' OR '1'='1' LIMIT 1--
query:    SELECT * FROM users WHERE username='' OR '1'='1' LIMIT 1-- ' AND password='...'
```

When only the password field is injectable and the username is fixed, commenting it out does not help (`admin'--` in the password just makes the test `password='admin'`). Use a condition that re-selects the target instead, so the trailing `AND password=...` is satisfied by an `OR`:

```
password: ' OR username='admin'--
query:    SELECT * FROM users WHERE username='admin' AND password='' OR username='admin'--'
```

Because `AND` binds tighter than `OR`, this evaluates as `(username='admin' AND password='') OR username='admin'`, which returns the admin row regardless of the password. Because PostgreSQL permits stacked queries more often than MySQL, a login injection point can sometimes do more than bypass, for example appending `; UPDATE users SET password=...` where the driver allows it, but the bypass itself needs only the row to be returned.

## Tools

- **sqlmap**: automated detection of the injectable login parameter.
- **Burp Repeater**: craft comment-termination and OR-based login payloads by hand.

## References

- PostgreSQL Documentation: comments, SELECT, WHERE
- OWASP Testing Guide: Testing for SQL Injection
