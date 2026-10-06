---
title: "Fingerprinting and enumeration in IBM Db2 injection"
order: 4
description: "Orienting an IBM Db2 injection: confirming the engine, reading the service level and special registers, and mapping the schema through the SYSCAT catalog."
keywords:
  - Db2 fingerprinting
  - SYSIBMADM.ENV_INST_INFO
  - special registers
  - SYSCAT.TABLES
  - SYSIBM.SYSDUMMY1
---

# Enumeration

Confirm the engine is Db2 and read the context before choosing a technique. The need for `FROM SYSIBM.SYSDUMMY1` on single-row selects is itself a fingerprint, as is the `SYSCAT` catalog.

Read the service level and identity. The version and fixpack come from an administrative view, not a function:

```sql
' UNION SELECT SERVICE_LEVEL,FIXPACK_NUM FROM SYSIBMADM.ENV_INST_INFO-- 
' UNION SELECT CURRENT USER,CURRENT SERVER FROM SYSIBM.SYSDUMMY1-- 
```

`SYSIBMADM.ENV_SYS_INFO` adds host and OS details. The special registers give identity: `CURRENT USER` (the authorization ID for checks), `SESSION_USER` and `SYSTEM_USER` (the session and connected IDs), `CURRENT SCHEMA`, and `CURRENT SERVER` (the database).

Map the schema through `SYSCAT`. List tables, then columns:

```sql
' UNION SELECT TABNAME,TABSCHEMA FROM SYSCAT.TABLES-- 
' UNION SELECT COLNAME,TYPENAME FROM SYSCAT.COLUMNS WHERE TABNAME='USERS'-- 
```

Catalog names are stored upper-case, so match them in upper case. The older `SYSIBM.SYSTABLES` and `SYSIBM.SYSCOLUMNS` are alternatives when `SYSCAT` is filtered. Grants for the current authorization ID come from `SYSCAT.DBAUTH` and `SYSCAT.TABAUTH`, which the privileges page uses to decide what the rest of the attack can reach. With the service level, identity, and schema known, extraction proceeds with union, blind, or time-based techniques.

## Tools

- **sqlmap**: automated fingerprinting and schema enumeration (`--banner`, `--tables`, `--columns`).
- **ghauri**: fast alternative with strong WAF evasion.
- **db2** (or clpplus): the Db2 client for reading the special registers and `SYSCAT` catalog.

## References

- IBM Db2 SQL Reference: special registers, SYSIBMADM administrative views, SYSCAT catalog
- OWASP Testing Guide: Testing for SQL Injection
