---
title: "PostgreSQL error-based SQLi using CAST and ::type conversion failures"
description: Forcing invalid casts so version(), schema names, or substrings appear in SQL error text returned to the client.
keywords:
  - PostgreSQL SQL injection
  - CAST
---

# CAST errors

## Context

**CAST(x AS numeric)** when **x** is non-numeric surfaces **text** in the error. Useful when **UPDATEXML**-style XML errors are unavailable. Aligns with Library Structure **CAST** under PostgreSQL error-based.
