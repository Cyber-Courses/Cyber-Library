---
title: "Oracle Java class execution in database context"
description: Java stored procedures in Oracle—risk when definer rights and grants allow runtime.exec-style behavior from SQL contexts.
keywords:
  - java stored procedure
  - Oracle Java
---
# Java class (Oracle)

**Java** stored in the database can reach **`Runtime.exec`**-class behavior when JVM **policy** and **grants** allow—typically a **DBA-tier** story, but worth the check when your SQLi runs as a fat schema.

## Context

Java stored procedures can call `Runtime.exec` when JVM permissions allow—often a **`sys`/`SYSTEM`** class of issue.
## Technique

Load or invoke classes via SQL once **`CREATE PROCEDURE`** / Java perms exist.
## Practice

- Check `DBA_JAVA_POLICY` / modern ACL equivalents for your version.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Command execution (Oracle)](index.md)
