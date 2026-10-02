---
title: "Authentication bypass through Oracle injection"
description: "Bypassing a login whose SQL is built from the username and password fields in Oracle Database, using comment termination and always-true conditions."
keywords:
  - authentication bypass
  - login bypass
  - OR 1=1
  - comment injection
  - Oracle login
---

# Authentication bypass

A login that interpolates the submitted fields, such as `SELECT * FROM users WHERE username='$u' AND password='$p'`, authenticates on any returned row. Injecting into either field can return the target row without the correct password.

Commenting out the password check logs in as a chosen user. Oracle's `--` comment removes the rest of the line:

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

An always-true condition returns a row when no account is targeted. Oracle has no `LIMIT`, so constrain with `ROWNUM` if the application expects a single row:

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

When only the password field is injectable and the username is fixed, comment termination does not bypass (it would just test `password='admin'`); use a condition that re-selects the target so the trailing `AND` is satisfied by an `OR`:

```
password: ' OR username='admin'--
```

Because `AND` binds tighter than `OR`, this evaluates as `(username='admin' AND password='') OR username='admin'`, returning the admin row. Oracle does not allow stacked queries through the usual drivers, so a login injection cannot append a second statement, but the row-returning bypass needs only the one query.

## Tools

- **sqlmap**: confirms and fingerprints the injectable login field.
- Manual testing with Burp Repeater and the Oracle client (sqlplus or SQLcl).

## References

- Oracle Database SQL Language Reference: comments, SELECT, conditions
- OWASP Testing Guide: Testing for SQL Injection
