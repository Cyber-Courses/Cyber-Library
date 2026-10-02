---
title: "SQLAlchemy injection"
description: "Injection through raw execute() strings and text() fragments built with f-strings or concatenation."
keywords:
  - SQLAlchemy
  - text()
  - execute()
  - raw SQL
  - Python
---

# SQLAlchemy

SQLAlchemy's Core and ORM expression language parameterizes values, but raw strings passed to `execute()` and `text()` fragments built with f-strings or concatenation are injectable, most often where a developer appends a dynamic `WHERE`, `ORDER BY`, or `LIMIT` to an otherwise safe query.

## Tools

- **sqlmap**: exploiting raw execute() and text() string-built sinks.
- **Burp Repeater**: testing dynamic WHERE, ORDER BY, and LIMIT interpolation.

## References

- SQLAlchemy documentation: Using textual SQL
- OWASP: SQL Injection
