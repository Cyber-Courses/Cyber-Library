---
title: "Python ORM injection"
order: 2
description: "Django ORM and SQLAlchemy parameterize ordinary queries, but raw-SQL escape hatches, text clauses, and lookup or filter construction from untrusted input bypass that binding."
keywords:
  - Python ORM
  - Django ORM injection
  - SQLAlchemy injection
  - raw SQL
  - text clause
---

# Python

Python web applications query through Django's ORM or SQLAlchemy, both of which bind parameters on their normal query paths. Injection shows up at the deliberate escape hatches and at the places where a query is assembled from strings rather than expressions.

## The ORMs here

- **[Django ORM](django-orm/index.md)**: `raw()` and `cursor.execute()`, `extra()` select and where fragments, and lookup construction in `filter()`.
- **[SQLAlchemy](sqlalchemy/index.md)**: `text()` clauses and raw `execute()` built by string formatting rather than bound parameters.

## The shared pattern

Both ORMs are safe when the query is expressed through their API with bound parameters, and vulnerable when a developer formats a string. Django's `raw()`/`extra()` and SQLAlchemy's `text()` are the explicit raw surfaces; the subtler case is building a filter, a lookup, or an `ORDER BY` from input, where the structure (not just a value) comes from the caller. Identifiers (table and column names) are never parameterizable, so any ORM path that interpolates a user-chosen identifier is a sink regardless of binding.

## Tools

- **sqlmap**: exploiting Django raw()/extra() and SQLAlchemy text() sinks.
- **Burp Repeater and Intruder**: crafting raw-SQL payloads and lookup or filter abuse.

## References

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Django: Performing raw SQL queries](https://docs.djangoproject.com/en/stable/topics/db/sql/)
