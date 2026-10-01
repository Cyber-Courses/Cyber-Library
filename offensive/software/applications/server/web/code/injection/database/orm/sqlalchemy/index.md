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

SQLAlchemy's Core and ORM expression language parameterizes values, but raw strings passed to `execute()` and `text()` fragments built with f-strings or concatenation are injectable — most often where a developer appends a dynamic `WHERE`, `ORDER BY`, or `LIMIT` to an otherwise safe query.
