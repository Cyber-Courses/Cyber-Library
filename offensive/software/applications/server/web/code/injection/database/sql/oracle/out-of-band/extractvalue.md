---
title: "Oracle EXTRACTVALUE and out-of-band XML fetches"
description: XPath and XMLType surfaces that can trigger network resolution, ACLs and egress controls for XML-enabled schemas.
keywords:
  - EXTRACTVALUE
  - XMLType
  - XPath
---
# EXTRACTVALUE (Oracle)

**EXTRACTVALUE** and **XMLType** constructors can process **XML** that references **external** entities or URLs depending on parser settings and privileges, overlapping with **XXE**-class issues in **XML** pipelines.

## Context

`EXTRACTVALUE` on `XMLType` can be weaponized for OOB when entity resolution or URL fetches occur, version-dependent.
## Technique

Use Collaborator to see if the parser hits your endpoint.
## Practice

- Overlaps XXE-style issues, tag for combined XML injection findings.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
