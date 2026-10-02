---
title: "IBM Db2 hex literals and string obfuscation"
description: Hex notation and CHR-style assembly of strings, useful context for detection rules and code review.
keywords:
  - HEX
  - x literal
---
# Hex encoding (IBM Db2)

**Hex** **literals** and **CHR**/**CONCAT** **chains** can **assemble** **banned** **keywords** without writing them **verbatim** in **raw** input.

## Context

Assemble banned tokens with `CHR` chains or `VARCHAR(x'4142')` style literals.
## Technique

Use when signatures match literal `UNION` / `SELECT` tokens.
## Practice

- Combine with comment smuggling for layered WAFs.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
