---
title: "LDAP filter injection: parentheses, OR clauses, and wildcard completion"
description: Breaking out of LDAP filter strings with unescaped user input to broaden search results or bypass login binds.
keywords:
  - LDAP injection
---

# LDAP filter injection

## Context

Filters like `(uid=%s)` become `(uid=*)(uid=*))(|(uid=*` when **parentheses** and **boolean** operators are injected. **Wildcard** `*` in some positions matches **all** entries.

## Theory

Use **parameterized** LDAP filters or strict **escape** routines for DN and filter **assertion** values per RFC 4515.

## See also

- [LDAP injection (parent)](index.md)
