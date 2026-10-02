---
title: "IBM Db2 alternate SQL syntax for filter evasion"
description: CONCAT variants, CASE, and UNION NULL probes—documented for WAF rule authors, not as a substitute for binds.
keywords:
  - CONCAT
  - UNION SELECT NULL
---
# Alternate syntax (IBM Db2)

Attackers vary **keyword** **casing**, **comment** placement, and **function** choices to evade **signature** **rules**. **Db2**-specific **concatenation** and **cast** idioms differ from MySQL or PostgreSQL.

## Context

Rotate **`CONCAT`**, `CHR`, hex literals, and `UNION SELECT NULL,NULL` probes to slip past immature filters.
## Technique

Db2 tolerates varied quoting—fuzz keyword split points.
## Practice

- Automate with Burp payload lists tuned to Db2.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [WAF bypass (IBM Db2)](index.md)
