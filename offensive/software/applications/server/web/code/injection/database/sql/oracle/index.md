---
title: "Oracle SQL injection: PL/SQL, dictionary views, and XML surfaces"
description: Oracle Database–specific SQL injection patterns—union and blind techniques, UTL packages, scheduler abuse, and XML/XXE-relevant functions in application contexts.
keywords:
  - Oracle SQL injection
  - PL/SQL
  - dual
---
# Oracle Database (SQLi)

**Oracle** deployments combine SQL and **PL/SQL**, rich **data dictionary** views, and packages such as **`UTL_HTTP`** and **`UTL_INADDR`** for network-oriented behavior when privileges allow. The layout below follows the **Library Structure** topic tree for Oracle SQL (structure topic id 1978).

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## Techniques (overview)

| Topic | Path |
|-------|------|
| Blind | [Blind](blind/index.md) — boolean and time-based inference |
| Command execution | [Command execution](command-execution/index.md) — Java and job-related surfaces |
| Error-based | [Error-based](error-based.md) |
| Evasion techniques | [Evasion techniques](evasion-techniques.md) |
| File manipulation | [File manipulation](file-manipulation/index.md) |
| Listener and SID enumeration | [Listener and SID enumeration](listener-and-sid-enumeration.md) |
| Out of band | [Out of band](out-of-band/index.md) |
| PLSQL | [PLSQL](plsql.md) |
| Persistence techniques | [Persistence techniques](persistence-techniques.md) |
| Privilege escalation | [Privilege escalation](privilege-escalation.md) |
| Session and user enumeration | [Session and user enumeration](session-and-user-enumeration.md) |
| Union-based | [Union-based](union-based/index.md) |
| XML and XXE abuse | [XML and XXE abuse](xml-and-xxe-abuse.md) |

## See also

- [SQL injection (parent)](../index.md)
