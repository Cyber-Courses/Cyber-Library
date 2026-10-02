---
title: "SQLAlchemy text() injection: interpolating input into textual SQL clauses"
description: text() marks a raw SQL fragment; building that fragment with f-strings or concatenation instead of bound :params makes it injectable even inside otherwise-ORM code.
keywords:
  - SQLAlchemy
  - text() injection
  - textual SQL
  - bound parameters
  - Python ORM
  - SQL injection
---

# SQLAlchemy text() injection

`text()` wraps a literal SQL string so SQLAlchemy will execute it, and it supports **bound parameters** via `:name` placeholders. Injection happens when the string handed to `text()` is assembled from user input instead of using those placeholders, common when a developer adds a dynamic `WHERE`, `ORDER BY`, or `LIMIT` to an otherwise ORM-based query.

## Vulnerable patterns

```python
from sqlalchemy import text

# term from the request
stmt = text(f"SELECT * FROM products WHERE name LIKE '%{term}%'")
session.execute(stmt)

# dynamic ORDER BY — cannot be a bound parameter, so often interpolated
session.execute(text(f"SELECT * FROM products ORDER BY {sort}"))
```

The safe form is `text("... LIKE :term")` with `.params(term=...)`; the value then travels as a bound parameter.

## Exploitation

**Value contexts** behave like standard injection. For the `LIKE` string above:

```
%' OR '1'='1
%' UNION SELECT username, password, NULL FROM users --
```

**`ORDER BY` / identifier contexts** cannot be parameterized, so they remain injectable even in disciplined codebases. Use them for inference and subquery-in-sort tricks:

```
(CASE WHEN (SELECT 1 FROM users WHERE username='admin' AND SUBSTRING(password,1,1)='a') THEN 1 ELSE 2 END)
```

Mixed usage is the classic trap: a query that binds most values with `:params` but appends one interpolated clause is still fully exploitable through that clause. Match payloads to the configured backend dialect (PostgreSQL/MySQL/SQLite/etc.).

## References

- [SQLAlchemy docs: Using textual SQL](https://docs.sqlalchemy.org/en/20/core/connections.html#using-textual-sql)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
