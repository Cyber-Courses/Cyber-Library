---
title: "PostgreSQL stacked queries: semicolon-separated statements in injectable SQL"
description: When the driver allows multiple statements, attackers chain CREATE, COPY, or DO blocks—often blocked by APIs and ORMs.
keywords:
  - stacked queries
  - PostgreSQL SQL injection
---

# Stacked queries

## Context

**Stacked** execution (`;` between statements) depends on the **client library** and **protocol**. **PgJDBC** and many ORMs **disallow** multiple statements by default. Maps to Library Structure **Stacked Query**.

## See also

- [PostgreSQL (parent)](index.md)
