---
title: "SQLite load_extension and native code execution"
description: Native extension loading in SQLite, disable in production and audit custom builds.
keywords:
  - load_extension
  - SQLite RCE
---
# Load extension (SQLite)

**`load_extension`** loads a **shared library** implementing SQLite entry points. It is **equivalent** to **arbitrary code execution** in the **process** hosting SQLite.

## Context

`load_extension('/path/evil.so')` loads a shared object, native code execution in the app process.
## Technique

Requires compile-time enablement and runtime permission; many mobile builds disable it.
## Practice

- Pre-position a SO/DLL via another vuln, then call `load_extension` from SQLi.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
