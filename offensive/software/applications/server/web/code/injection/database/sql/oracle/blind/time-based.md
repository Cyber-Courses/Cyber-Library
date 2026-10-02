---
title: "Oracle time-based blind SQL injection: DBMS_LOCK and conditional delays"
description: Timing-based inference in Oracle when explicit sleep or wait primitives are available and network jitter allows measurement.
keywords:
  - DBMS_LOCK.SLEEP
  - DBMS_PIPE.RECEIVE_MESSAGE
  - Oracle time-based SQLi
---
# Time-based (Oracle)

**Time-based** channels use deliberate **waits** tied to boolean conditions—package calls such as **`DBMS_LOCK.SLEEP`** or **`DBMS_PIPE.RECEIVE_MESSAGE`** when executable by the session. Availability depends on **privileges** and **package grants**.

## Context

Oracle delays via **`DBMS_LOCK.SLEEP`**, **`DBMS_PIPE.RECEIVE_MESSAGE`**, or heavy PL/SQL depending on grants.
## Technique

Gate delay behind boolean: `CASE WHEN <bit> THEN dbms_lock.sleep(5) ELSE 0 END` inside injectable SQL.
## Practice

- Check `dba_tab_privs` for execute on `DBMS_LOCK` from your session user.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Blind (Oracle)](index.md)
