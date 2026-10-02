---
title: "Authentication bypass through MSSQL injection"
description: "Bypassing a login whose T-SQL is built from the username and password fields, using comment termination and always-true conditions in SQL Server."
keywords:
  - authentication bypass
  - login bypass
  - OR 1=1
  - comment injection
  - MSSQL login
---

# Authentication bypass

A login that interpolates the submitted fields, such as `SELECT * FROM users WHERE username='$u' AND password='$p'`, authenticates on any returned row. Injecting into either field can return the target row without the correct password.

Commenting out the password check logs in as a chosen user. SQL Server's `--` comment needs no trailing space, so `admin'--` closes the string and drops the rest of the line:

```
username: admin'--
query:    SELECT * FROM users WHERE username='admin'-- ' AND password='...'
```

The password test is commented out and the admin row is returned. An always-true condition returns the first row when no account is targeted:

```
username: ' OR 1=1--
query:    SELECT * FROM users WHERE username='' OR 1=1-- ' AND password='...'
```

When only the password field is injectable and the username is fixed, comment termination does not bypass (it would just test `password='admin'`); use a condition that re-selects the target so the trailing `AND` is satisfied by an `OR`:

```
password: ' OR username='admin'--
```

Because `AND` binds tighter than `OR`, this evaluates as `(username='admin' AND password='') OR username='admin'`, returning the admin row. Since MSSQL permits stacked queries, a login injection can sometimes go further (for example `; UPDATE users SET ...`), but the bypass itself needs only the row to be returned.

## References

- Microsoft SQL Server Documentation: comments, SELECT, operator precedence
- OWASP Testing Guide: Testing for SQL Injection
