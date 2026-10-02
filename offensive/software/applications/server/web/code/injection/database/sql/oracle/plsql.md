---
title: "PL/SQL injection in Oracle"
description: "Injecting into Oracle PL/SQL: anonymous blocks, dynamic SQL built with EXECUTE IMMEDIATE, and abusing definer-rights procedures to run code as their owner."
keywords:
  - PL/SQL injection
  - anonymous block
  - EXECUTE IMMEDIATE
  - definer rights
  - invoker rights
---

# PL/SQL injection

Because Oracle does not allow stacked queries through the usual drivers, multi-statement action comes from PL/SQL instead. Two situations expose it: an injectable anonymous block, and a stored procedure that builds dynamic SQL from input.

A procedure that concatenates input into `EXECUTE IMMEDIATE` is injectable just like a web query. If the vulnerable code runs `EXECUTE IMMEDIATE 'SELECT ... WHERE name=''' || p_in || ''''`, a payload closes the literal and appends logic. The decisive factor is whose rights the code runs with.

A **definer-rights** procedure (the default, `AUTHID DEFINER`) executes with the privileges of its owner, not the caller. Injecting into a definer-rights procedure owned by a high-privileged schema (historically several `SYS`-owned packages) runs the injected SQL as that owner, which is the classic Oracle privilege-escalation path. An **invoker-rights** procedure (`AUTHID CURRENT_USER`) runs as the caller, so it offers no escalation but is still injectable for data access.

Where the injection point is itself a PL/SQL block, an attacker can run a full block:

```sql
'; BEGIN EXECUTE IMMEDIATE 'GRANT DBA TO '||USER; END;-- 
```

Inside a definer-rights `SYS` context, that `GRANT` succeeds and makes the current user a DBA. The same `EXECUTE IMMEDIATE` runs DDL, calls packages, and creates objects, so PL/SQL injection is the pivot from a data-read bug to full database control. Finding which reachable procedure is definer-rights and over-privileged is the key step.

## Tools

- **sqlmap**: detects injectable parameters that reach dynamic SQL.
- **ODAT** (Oracle Database Attacking Tool): abuses definer-rights procedures and PL/SQL for escalation.
- Manual testing with Burp Repeater and the Oracle client (sqlplus or SQLcl).

## References

- Oracle Database PL/SQL Language Reference: dynamic SQL, invoker's and definer's rights
- OWASP Testing Guide: Testing for SQL Injection
