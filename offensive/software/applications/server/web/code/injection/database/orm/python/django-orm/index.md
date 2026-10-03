---
title: "Django ORM injection"
description: "Injection through Django's raw(), extra(), the low-level cursor, and user-controlled filter() lookups."
keywords:
  - Django ORM
  - raw()
  - extra()
  - filter() injection
  - Python
---

# Django ORM

Django's ORM parameterizes normal querysets, but several APIs reopen injection when built from untrusted input: `raw()` and the low-level `connection.cursor()`, the legacy `extra()` with its `select`/`where` fragments, and user-controlled field or lookup names passed into `filter()`.

## Pages

- **[extra() injection](extra-injection.md)**: QuerySet.extra() splices raw SQL fragments into select, where, tables, and order_by, when those fragments carry user input the result is SQL injection.
- **[filter() injection](filter-injection.md)**: Expanding attacker-controlled dictionaries into QuerySet.filter()/exclude() and trusting user-supplied field lookups exposes unintended columns and boolean l...
- **[Raw SQL injection](raw-sql-injection.md)**: How string-formatted queries in Django's raw(), extra(), and the low-level cursor API become SQL injection despite the ORM, with concrete payloads.

## Tools

- **sqlmap**: exploiting raw() and cursor.execute() string-built sinks.
- **Burp Repeater and Intruder**: testing extra() fragments and user-controlled filter lookups.

## References

- Django documentation: Performing raw SQL queries
- OWASP: SQL Injection
