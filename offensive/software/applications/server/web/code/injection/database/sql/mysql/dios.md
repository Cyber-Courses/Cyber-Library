---
title: "Dump in one shot (DIOS) for MySQL injection"
order: 3
description: "Building a single MySQL union payload that concatenates schema and data into one response with nested subqueries and GROUP_CONCAT."
keywords:
  - DIOS
  - dump in one shot
  - GROUP_CONCAT
  - single request extraction
  - MySQL mass extraction
---

# Dump in one shot

Dump in one shot (DIOS) packs an entire enumeration into a single union payload, so one request returns the schema (and often the data) instead of a separate request per table or column. It trades a long, fragile payload for far fewer round trips, which matters against rate limits or slow blind channels.

The core is a subquery that walks `information_schema.columns` and uses `GROUP_CONCAT` to flatten every `table.column` into one string, dropped into a reflected position:

```sql
' UNION SELECT NULL,(SELECT GROUP_CONCAT(table_name,0x2e,column_name SEPARATOR 0x0a) FROM information_schema.columns WHERE table_schema=database()),NULL-- 
```

More elaborate DIOS payloads add formatting (HTML line breaks, delimiters) so the dump is readable in the rendered page, and some nest a second `GROUP_CONCAT` to pull actual row data for discovered tables in the same request.

The practical limit is `group_concat_max_len` (1024 bytes by default), which truncates large dumps. Raise it with `SET SESSION group_concat_max_len=1000000` where the account allows, or fall back to paged extraction for tables that overflow. DIOS is a convenience built on the same `information_schema` and `GROUP_CONCAT` primitives as ordinary union extraction, not a separate vulnerability.

## Tools

- **sqlmap**: automated union extraction that replaces hand-built DIOS payloads.
- **Burp Repeater**: tune and replay the single-request DIOS payload manually.

## References

- MySQL Reference Manual: GROUP_CONCAT, `group_concat_max_len`, `information_schema`
- OWASP Testing Guide: Testing for SQL Injection
