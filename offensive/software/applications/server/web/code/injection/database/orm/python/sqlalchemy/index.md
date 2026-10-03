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

## Pages

- **[Raw SQL injection](raw-sql-injection.md)**: Passing concatenated strings to Connection/Session.execute() or Engine.execute() bypasses SQLAlchemy's bound parameters and produces SQL injection.
- **[text() injection](text-injection.md)**: text() marks a raw SQL fragment; building that fragment with f-strings or concatenation instead of bound :params makes it injectable even inside otherwise-OR...

## Tools

- **sqlmap**: exploiting raw execute() and text() string-built sinks.
- **Burp Repeater**: testing dynamic WHERE, ORDER BY, and LIMIT interpolation.

## References

- SQLAlchemy documentation: Using textual SQL
- OWASP: SQL Injection
