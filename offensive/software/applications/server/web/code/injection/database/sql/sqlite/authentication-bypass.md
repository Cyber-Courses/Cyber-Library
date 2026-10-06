---
title: "Authentication bypass through SQLite injection"
order: 8
description: "Bypassing a login whose SQL is built from the username and password fields in SQLite, using comment termination and always-true conditions."
keywords:
  - authentication bypass
  - login bypass
  - OR 1=1
  - comment injection
  - SQLite login
---

# Authentication bypass

A login that interpolates the submitted fields, such as `SELECT * FROM users WHERE username='$u' AND password='$p'`, authenticates on any returned row. SQLite is a common back end for small apps and mobile and desktop software, where this pattern is frequent.

Commenting out the password check logs in as a chosen user. SQLite's `--` comment removes the rest of the line:

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

An always-true condition returns the first row when no account is targeted:

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

When only the password field is injectable and the username is fixed, comment termination does not bypass (it would just test `password='admin'`); re-select the target so the trailing `AND` is satisfied by an `OR`:

```
password: ' OR username='admin'--
```

Because `AND` binds tighter than `OR`, this evaluates as `(username='admin' AND password='') OR username='admin'`, returning the admin row. Whether a stacked statement can be appended depends on the API (`sqlite3_exec` allows it, a prepared statement does not), but the bypass needs only the one query to return a row.

## Tools

- **sqlmap**: confirms and fingerprints the injectable login field.
- Manual testing with Burp Repeater and the sqlite3 client.

## References

- SQLite Documentation: comments, SELECT, expressions
- OWASP Testing Guide: Testing for SQL Injection
