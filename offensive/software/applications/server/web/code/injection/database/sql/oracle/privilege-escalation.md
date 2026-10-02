---
title: "Oracle privilege escalation: DBA roles, grants, and Java privileges"
description: Role and grant abuse in Oracle, why application schemas must start minimal and how dangerous privileges are audited.
keywords:
  - GRANT DBA
  - CREATE ANY JOB
  - Oracle privilege escalation
---
# Privilege escalation (Oracle)

**DBA** and **`CREATE ANY`**-style privileges let principals alter **users**, **roles**, and **objects** across the database. **SQL injection** running as a **powerful** schema is near **full compromise**.

## Context

Oracle privesc chains: `GRANT DBA`, abuse `CREATE ANY JOB`, Java privileges, or become default `SYS` operations through weak ACLs.
## Technique

Start from current user → role tab → `DBA_ROLE_PRIVS` / `USER_SYS_PRIVS` for paths to `DBA`.
## Practice

- Java and scheduler abuse often needs packages your user can execute, enumerate grants aggressively.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
