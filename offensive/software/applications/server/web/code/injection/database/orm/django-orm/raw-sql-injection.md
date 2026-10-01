---
title: "Django ORM raw SQL injection: raw(), extra(), and cursor.execute() string building"
description: How string-formatted queries in Django's raw(), extra(), and the low-level cursor API become SQL injection despite the ORM, with concrete payloads.
keywords:
  - Django ORM
  - raw SQL injection
  - raw()
  - cursor.execute()
  - django.db.connection
  - SQL injection
---

# Raw SQL injection

Django's ORM parameterizes normal queryset operations, but it also exposes **escape hatches** that hand raw SQL to the database: `Model.objects.raw()` and the low-level `connection.cursor()` API. When application code builds the SQL string for either with Python string formatting instead of **parameter placeholders**, the ORM provides no protection and the sink is classic SQL injection.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable patterns

`raw()` with an f-string or `%`/`.format()` interpolation:

```python
# user_id comes straight from the request
User.objects.raw(f"SELECT * FROM auth_user WHERE id = {user_id}")
User.objects.raw("SELECT * FROM auth_user WHERE name = '%s'" % name)
```

The low-level cursor with the same mistake:

```python
with connection.cursor() as cur:
    cur.execute("SELECT * FROM auth_user WHERE username = '" + username + "'")
```

The safe form passes `params=[...]`/`%s` placeholders; the injectable form puts attacker text directly into the query string.

## Exploitation

Because the value lands inside a normal SQL statement, standard injection applies. For a numeric `id` context:

```
1 OR 1=1
0 UNION SELECT id, username, password FROM auth_user --
```

For the single-quoted `name`/`username` context, break out of the string first:

```
' OR '1'='1
' UNION SELECT username, password, NULL FROM auth_user --
```

Column count and types must match the original `SELECT` for a `UNION` to succeed; enumerate with `ORDER BY n` or incremental `NULL` columns as with any union-based injection. Django runs on PostgreSQL, MySQL, SQLite, and Oracle, so tailor comment syntax (`--`, `#`) and string functions to the backend in use.

Stacked queries are generally **not** available through the default DB-API cursor (one statement per `execute`), so prefer `UNION` and boolean/time-based inference over `; DROP ...`.

## Notes on `raw()` specifics

`raw()` maps rows to model instances, so the first columns must align with the model's primary key and fields for the rows to map—a `UNION` payload should select columns in the model's order, padding with `NULL`. Extra trailing columns can be pulled out through annotated attributes.

## References

- [Django docs: Performing raw SQL queries](https://docs.djangoproject.com/en/stable/topics/db/sql/)
- [Django docs: SQL injection protection](https://docs.djangoproject.com/en/stable/topics/security/#sql-injection-protection)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
