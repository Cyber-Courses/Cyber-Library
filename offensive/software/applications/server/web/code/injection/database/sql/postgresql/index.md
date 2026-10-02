---
title: "PostgreSQL SQL injection: Library Structure map—blind, error, time, file, command, privileges"
description: Engine-specific SQL injection aligned with the Library Structure PostgreSQL subtree (topic id 1943 in the structure database).
keywords:
  - PostgreSQL SQL injection
  - SQL injection
---

# PostgreSQL (SQLi)

This subtree follows the **Library Structure** topic tree for **PostgreSQL** under **SQL** (blind → boolean-based leaves, error-based → CAST/XML, command execution leaves, single-page topics for file, OOB, privileges, stacked query, time-based, WAF bypass). **Union-based** remains a practical **application** pattern and is kept as an extra hub.

Portable SQL notes live under [SQL (parent)](../index.md).

## Topics

| Topic | Path |
|-------|------|
| Blind | [Blind](blind/index.md) → [Boolean based](blind/boolean-based/index.md) |
| Command execution | [Command execution](command-execution/index.md) |
| Error-based | [Error-based](error-based/index.md) |
| File manipulation | [File manipulation](file-manipulation.md) |
| Out of band | [Out of band](out-of-band.md) |
| Privileges | [Privileges](privileges.md) |
| Stacked query | [Stacked query](stacked-query.md) |
| Time-based | [Time-based](time-based.md) |
| WAF bypass | [WAF bypass](waf-bypass.md) |
| Union-based (app layer) | [Union-based](union-based/index.md) |

## See also

- [SQL (parent)](../index.md)
- [MySQL](../mysql/index.md)
