---
title: "MSSQL role escalation to sysadmin: ALTER SERVER ROLE and server principals"
description: How membership in server roles like sysadmin can be granted, and why application accounts must never hold this capability.
keywords:
  - sysadmin
  - sp_addsrvrolemember
  - ALTER SERVER ROLE
---
# Make user DBA (MSSQL)

**sysadmin** is the highest **server role**. Granting it (historically via **`sp_addsrvrolemember`**, modernly via **`ALTER SERVER ROLE ... ADD MEMBER`**) should be **rare** and **audited**. Application connectivity must **not** use such accounts.

## Context

Escalation to **sysadmin** via `ALTER SERVER ROLE serveradmin ADD MEMBER` style ops, requires existing high privilege, not typical web users.
## Technique

If you already hold `securityadmin`/`sysadmin`, add your low-priv login or enable dormant principals.
## Practice

- Document role change in findings; this is critical-severity persistence.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
