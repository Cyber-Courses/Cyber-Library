---
title: "LDAP injection in application code: filter concatenation and bind DN assembly"
description: Untrusted input in LDAP search filters, DN strings, or attribute lists when the application builds directory queries as string concatenation.
keywords:
  - LDAP injection
  - directory service
---

# LDAP injection

LDAP injection appears where **filter** strings or **distinguished names** are built from **concatenated** fragments so attacker input lands **inside** the filter grammar. Closing parentheses and boolean operators can change the filter’s logic so the query matches **more** entries than the developer intended—classic auth bypass and data exfil patterns.

## Typical sinks

- Login forms that map `uid=` + user input + closing paren in a search filter.
- Admin tools that build `(|(cn=...)(mail=...))` from partial user input without escaping.

## Pages

| Page | Focus |
|------|--------|
| [LDAP filter injection](ldap-filter-injection.md) | Parentheses and boolean abuse |

## See also

- [Database injection (parent)](../index.md)
- [SQL](../sql/index.md)
