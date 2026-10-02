---
title: "Oracle Database SQL injection: techniques and payloads"
description: "Exploiting SQL injection against Oracle Database: the mandatory FROM DUAL, ACL-gated UTL packages, DBMS_PIPE timing, PL/SQL blocks, and the ALL_ catalog views."
keywords:
  - Oracle SQL injection
  - DUAL
  - UTL_HTTP UTL_INADDR
  - DBMS_PIPE
  - all_tables
  - PL/SQL injection
---

# Oracle

Oracle Database has the most distinctive dialect of the major engines, and several of its quirks decide how injection payloads are written.

Every `SELECT` needs a `FROM`, so single-row queries select `FROM DUAL`. There is no `LIMIT`; row limiting is `WHERE ROWNUM=1` or `FETCH FIRST n ROWS ONLY` (12c and later). Strings concatenate with `||`, and `CHR()` builds them without quotes. Comments are `--` and `/* */`. Crucially, Oracle does not support stacked queries through the usual JDBC and OCI drivers: a second `;`-separated statement does not run, so multi-statement actions require an injectable PL/SQL block instead.

The catalog is the `ALL_`/`USER_`/`DBA_` views (`all_tables`, `all_tab_columns`, `all_users`) and the `V$` performance views (`v$version`, `v$instance`). Identity and context come from `SELECT user FROM dual` and `SYS_CONTEXT('USERENV', ...)`.

Oracle's most powerful primitives live in supplied PL/SQL packages, and from 11g their network reach (`UTL_HTTP`, `UTL_INADDR`, `UTL_TCP`, `UTL_SMTP`, `HTTPURITYPE`) is gated by fine-grained Access Control Lists, so out-of-band and some error channels depend on an ACL grant as well as the package execute privilege. State these preconditions when a payload relies on them.

## Techniques

- **[Enumeration](enumeration.md)**: version, session context, and current user.
- **[Authentication bypass](authentication-bypass.md)**: subvert a login built from the credential fields.
- **[Union-based](union-based.md)**: append a `UNION SELECT ... FROM DUAL` with matching types.
- **[Error-based](error-based.md)**: leak values through ORA errors that echo input.
- **[Blind](blind.md)**: infer data from boolean response differences.
- **[Time-based](time-based.md)**: infer data with `DBMS_PIPE.RECEIVE_MESSAGE`.
- **[Privilege escalation](privilege-escalation.md)**: reach DBA through role and package abuse.
- **[PL/SQL injection](plsql.md)**: inject into anonymous blocks and dynamic SQL.
- **[File manipulation](file-manipulation.md)**: read and write files with `UTL_FILE` and directory objects.
- **[Out-of-band](out-of-band.md)**: exfiltrate over DNS and HTTP with the `UTL` packages.
- **[XML and XXE abuse](xml-and-xxe-abuse.md)**: abuse `XMLType` parsing for SSRF and file read.
- **[Command execution](command-execution.md)**: run OS commands via Java or `DBMS_SCHEDULER`.
- **[Listener and SID enumeration](listener-and-sid-enumeration.md)**: identify the instance and SID.
- **[Persistence](persistence-techniques.md)**: scheduler jobs, triggers, and backdoors.
- **[Evasion techniques](evasion-techniques.md)**: comments, `CHR()`, and case tricks past filters.

## References

- Oracle Database SQL Language Reference and PL/SQL Packages and Types Reference
- OWASP Testing Guide: Testing for SQL Injection
