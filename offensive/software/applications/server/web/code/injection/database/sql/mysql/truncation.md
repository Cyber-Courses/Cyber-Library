---
title: "MySQL truncation and username collision in SQL injection and registration logic"
description: VARCHAR length limits that create distinct usernames collapsing to the same stored value—auth edge case.
keywords:
  - truncation
  - MySQL SQL injection
---

# Truncation

## Context

Library Structure **Truncation**—**admin** + spaces vs **admin** collision when column length **truncates**. Pair with **registration** logic testing.

## See also

- [MySQL (parent)](index.md)
