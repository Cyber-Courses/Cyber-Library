---
title: "Union-based SQL injection (MySQL): column alignment, metadata, and data extraction"
description: UNION-based extraction in MySQL, column alignment, information_schema use, and blind column naming tricks.
keywords:
  - union SQL injection
  - MySQL union
  - information_schema
---

# Union-based SQLi

**Union-based** SQLi appends a `UNION SELECT` that adds rows to an existing result set. You must match **column count** and usually **types** with the original query’s projected columns. MySQL exposes **`information_schema`** for databases, tables, and columns when privileges allow.

## Pages

- [Detect column count](detect-column-count.md)
- [Extract database with information_schema](extract-database-with-information-schema.md)
- [Extract column names without information_schema](extract-column-names-without-information-schema.md)
- [Extract data without column names](extract-data-without-column-names.md)
