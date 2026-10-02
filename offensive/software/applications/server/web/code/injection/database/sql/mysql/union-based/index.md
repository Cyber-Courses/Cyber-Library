---
title: "Union-based SQL injection in MySQL: column alignment and data extraction"
description: "Using UNION SELECT in MySQL to append attacker-chosen rows to a query result, from column-count detection through information_schema extraction."
keywords:
  - union based SQL injection
  - UNION SELECT MySQL
  - column count detection
  - information_schema extraction
  - GROUP_CONCAT
---

# Union-based

When an injectable query returns its rows to the page, a `UNION SELECT` lets you append rows of your own choosing. The appended `SELECT` runs with the application's privileges and its columns appear wherever the original result is displayed, which turns a data-returning query into a general read primitive over everything the database account can reach.

Two conditions must hold. The injected `SELECT` has to project the **same number of columns** as the original query, and the columns you want to read must sit in positions whose **types are compatible** with what the page renders (usually a string column). `NULL` is type-compatible with everything, so it is the safe filler while you work out the layout.

The workflow is: detect the column count, find which columns are reflected, then read schema and data through those positions. MySQL's `information_schema` supplies the schema, and `GROUP_CONCAT()` collapses many rows into one so a single reflected cell can carry a whole table.

## Pages

- **[Detect column count](detect-column-count.md)**: `ORDER BY` and `UNION SELECT NULL` probing.
- **[Extract with information_schema](extract-with-information-schema.md)**: enumerate and dump via the catalog.
- **[Extract without information_schema](extract-without-information-schema.md)**: when the catalog is filtered.

## Tools

- **sqlmap**: automated union-based detection and extraction.
- **ghauri**: fast alternative with strong WAF evasion.

## References

- MySQL Reference Manual: UNION clause and `information_schema`
- OWASP Testing Guide: Testing for SQL Injection
