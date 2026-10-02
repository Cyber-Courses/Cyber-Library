---
title: "SQLite SQL injection evasion: comments, delimiters, and encoding"
description: Comment and delimiter tricks in SQLite—inline comments, semicolons, and null-byte edge cases in APIs that concatenate strings.
keywords:
  - SQLite comment
  - null byte
---
# Evasion techniques (SQLite)

SQLite accepts **`--`**, **`/* */`**, and **`;`** statement separators. Some **legacy** stacks mishandle **null** bytes or **multi-statement** strings. **Defense in depth** still requires **parameterization**.

## Context

SQLite accepts `--`, `/**/`, `;` statement separators—useful when filters key on spaces or keywords.
## Technique

Some APIs accidentally allow multi-statement strings—probe with benign second `SELECT`.
## Practice

- Try URL-encoding, double-encoding, and unicode homoglyphs against weak WAFs.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [SQLite (SQLi)](index.md)
