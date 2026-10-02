---
title: "Union-based SQL injection in PostgreSQL: NULL padding, :: casts, and information_schema"
description: UNION column alignment and metadata extraction using PostgreSQL catalogs, common in application-layer testing.
keywords:
  - PostgreSQL SQL injection
  - UNION
---

# Union based (PostgreSQL)

The Library Structure tree for **PostgreSQL** emphasizes blind, error, time, file, and privilege topics; **UNION** is still the workhorse for **verbose** injection in apps. Use **NULL::text** padding and **information_schema** the same way as portable SQL notes under [SQL (parent)](../../index.md).
