---
title: "Oracle Java execution grants: loadjava and DBMS_JAVA policy"
description: Operational controls for Java in Oracle—loadjava, permissions, and minimizing EXECUTE paths from SQL injection.
keywords:
  - loadjava
  - dbms_java.grant_permission
---
# Java execution (Oracle)

**Java execution** paths include **`loadjava`**, **`dbms_java`**, and **permission** grants inside the JVM embedded in the database. Misconfiguration can turn **SQL injection** into **arbitrary code** in the DB tier.

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
