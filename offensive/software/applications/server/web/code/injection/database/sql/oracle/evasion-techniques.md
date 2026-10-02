---
title: "Oracle SQL injection evasion: comments, encoding, and keyword obfuscation"
description: Comment-based and lexical tricks used to bypass naive filters, relevant to WAF tuning and safe parser design, not filter-only security.
keywords:
  - comment-based evasion
  - Oracle SQLi
---
# Evasion techniques (Oracle)

Attackers mix **`--`**, **`/**/`**, **`CHR`**, **`REPLACE`**, and **case** variation to slip past **blacklist** rules. **Evasion is not a root fix**; it shows why **parameterization** must be the primary control.

## Context

Filter evasion mixes comments, case, and encoding to slip past naive WAF regex, not a substitute for finding a real injectable parameter.
## Technique

Layer `/**/`, inline comments, `CHR` concatenation, and split keywords across case variants.
## Practice

- Automate mutations in Burp Intruder payload sets.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
