---
title: "PostgreSQL boolean blind SQLi with CASE WHEN: conditional bit tests in injected predicates"
description: Using CASE WHEN expressions in blind SQL injection to branch on substring comparisons against PostgreSQL catalogs or secrets.
keywords:
  - PostgreSQL SQL injection
  - CASE WHEN
  - blind SQLi
---

# CASE WHEN

## Context

`CASE WHEN condition THEN … ELSE … END` lets you express **bit tests** inside a single predicate when `IF()`-style wrappers are filtered. Aligns with Library Structure topic **Boolean with CASE WHEN** (PostgreSQL blind).

## Theory

Nest comparisons: `CASE WHEN substr(version(),1,1)='5' THEN true ELSE false END` adapted to your injectable fragment’s quoting.

## Practice

- Pair with application-visible differences (row present vs empty, 200 vs 500) in a staging database.

## See also

- [Boolean based (parent)](index.md)
