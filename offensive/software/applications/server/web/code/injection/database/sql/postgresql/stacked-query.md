---
title: "Stacked queries in PostgreSQL injection"
order: 7
description: "Running extra statements after a semicolon in PostgreSQL injection, which many PostgreSQL drivers permit, opening DDL, writes, and COPY from a single injection point."
keywords:
  - stacked queries
  - semicolon injection
  - multiple statements
  - PostgreSQL DDL injection
---

# Stacked queries

A stacked query appends a second, independent statement after a semicolon. PostgreSQL drivers permit this far more often than the common MySQL ones: `libpq`'s simple query protocol and PDO without prepared statements run every `;`-separated statement, so one injection point can execute arbitrary SQL, not just read.

Where the sink is a `SELECT`, append a statement that acts:

```sql
'; UPDATE users SET password='$2a$...' WHERE username='admin'-- 
'; CREATE TABLE x(d text)-- 
```

Stacked execution is what makes the strongest PostgreSQL primitives reachable from a read-only-looking query: creating a staging table and `COPY`-ing a file into it, writing a web shell with `COPY ... TO`, and running an OS command with `COPY ... FROM PROGRAM` all need a statement of their own, which stacking supplies.

Availability is driver-specific. Prepared-statement interfaces and drivers that send one statement per request reject the second statement, so stacking is confirmed by observing a side effect (a created table, an updated row) rather than assumed. When it is blocked, fall back to in-query techniques (union, error, blind) that work within the single original statement.

## Tools

- **sqlmap**: exploits stacked queries with `--technique=S`.
- **psql**: official client to confirm multi-statement execution.

## References

- PostgreSQL Documentation: multi-statement simple query, COPY
- OWASP Testing Guide: Testing for SQL Injection
