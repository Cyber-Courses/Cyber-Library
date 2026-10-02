---
title: "IBM Db2 command execution: ADMIN_CMD and external interfaces"
description: Administrative and stored-procedure paths that can reach host commands—strict separation of admin connectivity from application SQL.
keywords:
  - ADMIN_CMD
  - QCMDEXC
  - Db2 command execution
---
# Command execution (IBM Db2)

**Command** surfaces include **administrative** APIs (**`ADMIN_CMD`**, platform-specific **shell** bridges) reachable only from **privileged** accounts. **Application** logins should **never** invoke them.

## Context

Db2 command execution surfaces include **`ADMIN_CMD`**, **`QCMDEXC`** on IBM i contexts, and stored procedures—platform-specific.
## Technique

Requires elevated rights; map `SYSCAT.DBAUTH` / `SYSIBMADM` views from the injectable session.
## Practice

- Confirm OS family (LUW vs z/OS vs IBM i) before choosing payloads.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [IBM Db2 (SQLi)](index.md)
