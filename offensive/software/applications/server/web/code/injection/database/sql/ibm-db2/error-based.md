---
title: "Error-based SQL injection in IBM Db2"
description: "The narrow IBM Db2 error channel: using invalid casts and SIGNAL to leak values where possible, and why boolean or time-based inference is the dependable blind path."
keywords:
  - error based SQL injection
  - Db2 errors
  - invalid cast
  - SIGNAL SQLSTATE
  - SQLCODE
---

# Error-based

Error-based injection is weaker in Db2 than in PostgreSQL or SQL Server, because Db2's error messages do not reliably quote an arbitrary query result. It is most useful for confirming injection and fingerprinting, with the actual data read by inference.

An invalid conversion is the simplest error. Casting a non-numeric string to a number fails with an `SQL0420N`/`SQL0180`-type conversion error, which proves the point is live:

```sql
' AND 1=CAST((SELECT CURRENT USER FROM SYSIBM.SYSDUMMY1) AS INT)-- 
```

Where the value is non-numeric the cast fails; whether the failing value appears in the returned message depends on the driver and error verbosity, so this cannot be relied on to echo arbitrary data the way other engines do.

`SIGNAL` raises a custom error whose message text can carry a value, which is the closest Db2 offers to a value-echoing channel, though it needs a compound-statement or routine context rather than a plain injected expression. `MESSAGE_TEXT` accepts a simple value, not a scalar fullselect, so the result is first assigned to a declared variable and that variable is supplied:

```sql
BEGIN DECLARE v VARCHAR(128); SET v = (SELECT CURRENT SERVER FROM SYSIBM.SYSDUMMY1); SIGNAL SQLSTATE '75001' SET MESSAGE_TEXT = v; END
```

Because the message length is bounded and clean echoing is inconsistent, the practical approach against Db2 is to use errors to confirm and fingerprint (a distinct `SQLCODE`/`SQLSTATE` proves injection and can itself act as a boolean oracle), and to extract the data with the boolean and time-based techniques, which are the dependable channels here.

## References

- IBM Db2 SQL Reference: CAST, SIGNAL, SQLSTATE and SQLCODE values
- OWASP Testing Guide: Testing for SQL Injection
