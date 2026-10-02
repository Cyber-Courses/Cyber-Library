---
title: "Oracle persistence via database objects: jobs, triggers, and procedures"
description: Long-lived footholds in Oracle—scheduler jobs, triggers, and malicious packages for sustained access during engagements.
keywords:
  - DBMS_SCHEDULER
  - trigger backdoor
  - Oracle persistence
---
# Persistence techniques (Oracle)

Attackers with **DDL** or **job** privileges may install **triggers**, **jobs**, or **packages** that survive **session** end. **Persistence** is a **privilege** problem first.

## Context

Persistence installs DB jobs, triggers, or packages that survive the web session—useful for long assessments with intermittent access.
## Technique

`DBMS_SCHEDULER` jobs, malicious triggers on `AFTER LOGON`, or extra procedures granted to `PUBLIC`.
## Practice

- Clean up in out-of-scope prod; keep artifacts labeled in lab.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](index.md)
- [File manipulation](file-manipulation/index.md)
