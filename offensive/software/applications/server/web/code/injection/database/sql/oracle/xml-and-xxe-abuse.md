---
title: "Oracle XML and XXE abuse: EXTRACTVALUE, XMLTABLE, and external entities"
description: XML features in Oracle that overlap with XXE—entity resolution, external DTDs, and network egress from XML pipelines.
keywords:
  - EXTRACTVALUE
  - XMLTABLE
  - XXE
---
# XML and XXE abuse (Oracle)

Oracle provides **`EXTRACTVALUE`**, **`XMLType`**, **`XMLQUERY`**, and **`XMLTABLE`**. When **external** entities or **HTTP** fetches are enabled, **XML** injection can resemble **XXE** in application layers.

## Context

Oracle XML (`XMLType`, `EXTRACTVALUE`, `XMLTABLE`) can pull remote URLs or resolve entities depending on parser settings and ACLs.
## Technique

Chain external DTDs or `UTL_HTTP` feeds inside XML constructors for OOB or file read analogues.
## Practice

- Test in lab for network egress from DB tier before promising impact.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](index.md)
- [Out of band](out-of-band/index.md)
