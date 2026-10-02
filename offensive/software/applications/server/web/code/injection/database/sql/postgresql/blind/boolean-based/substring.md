---
title: "PostgreSQL boolean blind SQLi with SUBSTRING and SUBSTR: character-by-character extraction"
description: Bit-by-bit string inference using substr(), substring(), ascii(), and length() in PostgreSQL.
keywords:
  - PostgreSQL SQL injection
  - SUBSTRING
  - blind SQLi
---

# Substring comparison

## Context

**SUBSTRING(x FROM i FOR 1)** and **substr** extract one character for comparison to ASCII ranges or literals—standard blind workflow. Maps to Library Structure **Boolean with Substring**.

## See also

- [Boolean based (parent)](index.md)
