---
title: "SQLAlchemy raw SQL injection: execute() and string-built statements"
description: Passing concatenated strings to Connection/Session.execute() or Engine.execute() bypasses SQLAlchemy's bound parameters and produces SQL injection.
keywords:
  - SQLAlchemy
  - raw SQL injection
  - execute()
  - engine.execute
  - Python ORM
  - SQL injection
---

# Raw SQL injection

SQLAlchemy's Core and ORM layers parameterize queries built with its expression language, but applications frequently drop to raw strings via `connection.execute()`, `engine.execute()`, or `session.execute()`. When the statement string is concatenated or f-string-formatted with user input, SQLAlchemy sends it as-is and the sink is SQL injection.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable patterns

```python
# name from the request
conn.execute("SELECT * FROM users WHERE name = '" + name + "'")
engine.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

Modern SQLAlchemy requires a `text()` clause for raw SQL (see [Text() Injection](text-injection.md)), but the same concatenation mistake inside `text()` is equally injectable. The safe form binds parameters (`text("... = :name")` with `{"name": name}`).

## Exploitation

Exploit as ordinary injection in the value's context. Numeric `id`:

```
1 OR 1=1
0 UNION SELECT id, username, password FROM users
```

Quoted `name`:

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM users --
```

SQLAlchemy runs on PostgreSQL, MySQL, SQLite, Oracle, and SQL Server, so match the dialect: `--`/`#` comments, `pg_sleep()`/`SLEEP()` for time-based blind, and backend-specific string functions for substring extraction. The DBAPI driver usually permits one statement per `execute()`, so favor `UNION` and inference over stacked queries unless the driver is configured otherwise (e.g. `psycopg2` with multiple statements).

## References

- [SQLAlchemy docs: Working with raw SQL (text())](https://docs.sqlalchemy.org/en/20/core/connections.html#using-textual-sql)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
