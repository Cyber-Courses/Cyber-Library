---

## title: "IBM Db2 session and user metadata: SESSION_USER and authorization IDs"
description: Session-level identifiers in Db2—SESSION_USER, SYSTEM_USER—for orienting SQLi chains and impersonation paths.
keywords:
  - SESSION_USER
  - CURRENT USER

# Session and user information (IBM Db2)

`**SESSION_USER**`, `**SYSTEM_USER**`, and `**CURRENT USER**` (where supported) describe **who** runs the **SQL**. Use them to orient **impersonation** targets and to sanity-check whether you are on a **shared pool** account vs a named app user.

## Context

`SESSION_USER`, `CURRENT USER`, `SYSTEM_USER` orient the session for impersonation chains.

## Technique

Cross-check `SYSCAT.DBAUTH` for unexpected grants.

## Practice

- Document effective user in every finding appendix.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Enumeration techniques (IBM Db2)](index.md)