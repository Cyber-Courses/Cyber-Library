---
title: "PostgreSQL boolean blind SQLi with NULLIF and COALESCE: null-handling predicates"
description: Blind inference using NULLIF and COALESCE when simpler function names are blocked by filters.
keywords:
  - PostgreSQL SQL injection
  - NULLIF
  - COALESCE
---

# NULLIF and COALESCE

## Context

**NULLIF(x,x)** yields NULL; **COALESCE(NULL,1)** yields 1. These build **conditional nullness** useful when keyword filters block other primitives. Maps to Library Structure **Boolean with NULLIF COALESCE**.

## See also

- [Boolean based (parent)](index.md)
