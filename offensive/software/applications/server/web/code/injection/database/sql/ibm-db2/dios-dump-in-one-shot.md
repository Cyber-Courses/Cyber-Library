---
title: "IBM Db2 DIOS-style bulk extraction: XMLAGG and XMLROW patterns"
description: Concatenating many rows in a single response using XML aggregation helpers—useful for understanding data-exfil patterns and row limits.
keywords:
  - XMLAGG
  - XMLROW
  - DIOS
---
# DIOS dump in one shot (IBM Db2)

**DIOS**-style payloads **aggregate** multiple rows into **one** scalar using **`XMLAGG`**, **`XMLROW`**, or similar—reducing round trips for an attacker and increasing **payload** size in logs.

## Context

Bulk exfil in one response using **`XMLAGG`**, **`XMLROW`**, or concatenation—minimizes round trips for wide tables.
## Technique

Watch row/LOB limits and client truncation in HTTP responses.
## Practice

- Tune delimiter characters that survive the app’s HTML encoding.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [IBM Db2 (SQLi)](index.md)
