---
title: "IBM Db2 SQL injection: techniques and payloads"
description: "Exploiting SQL injection against IBM Db2 (LUW): the SYSIBM.SYSDUMMY1 single-row table, SYSCAT catalog, special registers, LISTAGG extraction, and the lack of a SLEEP function."
keywords:
  - Db2 SQL injection
  - SYSIBM.SYSDUMMY1
  - SYSCAT
  - special registers
  - LISTAGG
  - FETCH FIRST
---

# IBM Db2

IBM Db2 (the LUW edition on Linux, Unix, and Windows) has a distinctive dialect, and this page's payloads target it rather than Db2 for z/OS or Db2 for i, which differ.

Like Oracle, every `SELECT` needs a `FROM`, and the single-row pseudo-table is `SYSIBM.SYSDUMMY1` (Db2's `DUAL`). There is no `LIMIT`; row limiting is `FETCH FIRST n ROWS ONLY`. Strings concatenate with `||` or `CONCAT()`, and comments are `--` and `/* */`. Identity comes from the special registers `CURRENT USER`, `SESSION_USER`, `SYSTEM_USER`, `CURRENT SCHEMA`, and `CURRENT SERVER`.

The catalog lives in the `SYSCAT` views (`SYSCAT.TABLES` with `TABSCHEMA`/`TABNAME`, `SYSCAT.COLUMNS` with `COLNAME`, `SYSCAT.DBAUTH` for grants) and the older `SYSIBM.SYSTABLES`. Version and service level come from `SYSIBMADM.ENV_INST_INFO` and `SYSIBMADM.ENV_SYS_INFO`, not a `version()` function. Aggregation into one cell uses `LISTAGG` (Db2 9.7 and later) or `XMLAGG`.

Two limits shape technique choice. Db2 has no `SLEEP`/`WAITFOR`, so time-based inference relies on a deliberately heavy query, and its error messages rarely echo an arbitrary query result, so error-based is weak and boolean or time-based inference is the dependable blind channel. Stacked queries are generally unavailable through the standard CLI/JDBC drivers.

## Techniques

- **[Enumeration](enumeration.md)**: service level, special registers, and the SYSCAT catalog.
- **[Authentication bypass](authentication-bypass.md)**: subvert a login built from the credential fields.
- **[Union-based](union-based.md)**: append a `UNION SELECT ... FROM SYSIBM.SYSDUMMY1`.
- **[Error-based](error-based.md)**: the narrow Db2 error surface and what it can leak.
- **[Blind](blind.md)**: infer data from boolean response differences.
- **[Time-based](time-based.md)**: infer data with a heavy query, since Db2 has no SLEEP.
- **[Privileges](privileges.md)**: read authorities and grants from SYSCAT.DBAUTH.
- **[Command execution](command-execution.md)**: external routines, and the limits of ADMIN_CMD.
- **[DIOS](dios.md)**: single-request dumps with `XMLAGG`/`LISTAGG`.
- **[WAF bypass](waf-bypass.md)**: `CHR()`, concatenation, and hex past filters.

## Tools

- **sqlmap**: automated detection and exploitation (`--dbms=Db2`).
- **ghauri**: fast alternative with strong WAF evasion.
- **db2** (or clpplus): the Db2 client for a direct session once credentials are recovered.

## References

- IBM Db2 SQL Reference: special registers, catalog views, built-in functions
- OWASP Testing Guide: Testing for SQL Injection
