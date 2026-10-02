---
title: "IBM Db2 schema discovery: SYSIBM tables and SYSCAT views"
description: Enumerating tables, columns, and privileges through Db2 catalogs—scope SELECT grants carefully for application roles.
keywords:
  - SYSIBM.SYSTABLES
  - SYSCAT.TABLES
---
# Schema and metadata discovery (IBM Db2)

**`SYSIBM`** and **`SYSCAT`** views expose **schemas**, **tables**, **columns**, and **privileges**. Overly broad **SELECT** on **catalogs** aids attackers mapping the **database**.

## Context

`SYSCAT.TABLES`, `SYSCAT.COLUMNS`, `SYSIBM.SYSTABLES` map schema for union targets and blind guesses.
## Technique

Filter by `TABSCHEMA` to stay inside the app schema first.
## Practice

- Use metadata to prioritize high-value tables (users, tokens).

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Enumeration techniques (IBM Db2)](index.md)
