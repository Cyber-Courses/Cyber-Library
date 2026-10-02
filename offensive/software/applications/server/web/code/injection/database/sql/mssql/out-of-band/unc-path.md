---
title: "MSSQL UNC paths in SQL injection: xp_dirtree, backups, and NTLM context"
description: UNC-style paths that cause SQL Server to reach SMB resources, relevant to coercion, hash capture scenarios, and operational network segmentation.
keywords:
  - xp_dirtree
  - UNC
  - NTLM
  - MSSQL
---
# UNC path (MSSQL)

**UNC** paths (`\\server\share`) can force the SQL service account to **touch SMB**, `xp_dirtree`, `xp_fileexist`, **`BACKUP`/`RESTORE`**-style arguments, and similar. In **authorized** Windows/SMB lab setups, that can surface **NetNTLM** material for relay or offline cracking narratives; document what the engagement rules actually allow you to capture.

## Context

UNC paths make the SQL process touch `\\host\share`; Windows may attempt SMB and leak NetNTLM hashes to a listener in some lab setups.
## Technique

Call procedures that accept paths (`xp_dirtree`, `xp_fileexist`, backup APIs) with attacker-controlled UNC.
## Practice

- Use only in **owned** lab or explicit hash-capture engagements with written approval.
- Pair with **NTLM relay** / **Responder**-style narratives only where the engagement explicitly allows hash or relay work, document what you actually captured.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
