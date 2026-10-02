---
title: "Oracle PL/SQL injection: anonymous blocks, EXECUTE IMMEDIATE, and definer rights"
description: PL/SQL-specific injection—dynamic SQL, definer versus invoker rights, and procedure boundaries in application schemas.
keywords:
  - PL/SQL injection
  - EXECUTE IMMEDIATE
  - definer rights
---
# PLSQL (Oracle)

**PL/SQL** adds **procedural** boundaries: **triggers**, **packages**, and **`EXECUTE IMMEDIATE`**. Injection that reaches **dynamic SQL** inside PL/SQL may run with **definer** privileges if procedures are mis-scoped.

## Context

PL/SQL injection hits dynamic SQL inside procedures—`EXECUTE IMMEDIATE` with attacker-controlled fragments runs as **definer** if the proc is definer-rights.
## Technique

Chain nested quotes to break out of string literals inside PL/SQL blocks passed through the app.
## Practice

- Read package source (`ALL_SOURCE`) when accessible to find dynamic SQL sinks.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](index.md)
