---
title: "DynamoDB PartiQL injection: breaking out of ExecuteStatement string concatenation"
order: 1
description: "PartiQL statements run via ExecuteStatement and built by string concatenation are injectable like SQL: break out of the WHERE value, widen with OR, and read other items."
keywords:
  - PartiQL injection
  - DynamoDB
  - ExecuteStatement
  - WHERE clause injection
  - NoSQL injection
  - parameterized statements
---

# PartiQL injection

PartiQL is DynamoDB's SQL-compatible query language, executed through the `ExecuteStatement`, `BatchExecuteStatement`, and `ExecuteTransaction` APIs. Because the syntax is SQL-like (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `WHERE`, `AND`/`OR`), developers coming from relational databases build statements the same way they always have (with string formatting) and reintroduce the same injection class. The parameterized form uses `?` placeholders supplied in a separate `Parameters` list; injection happens when input is concatenated into the statement text instead.

## Vulnerable pattern

```python
# user_id and category come from the request
stmt = (
    "SELECT * FROM Orders "
    f"WHERE user_id = '{user_id}' AND category = '{category}'"
)
client.execute_statement(Statement=stmt)
```

The safe form keeps the statement static and passes values out of band:

```python
client.execute_statement(
    Statement="SELECT * FROM Orders WHERE user_id = ? AND category = ?",
    Parameters=[{"S": user_id}, {"S": category}],
)
```

## Exploitation

PartiQL quotes string literals with **single quotes** and identifiers with double quotes, so the break-out primitive is the single quote, exactly as in SQL.

**Widen the scope with `OR`.** Close the attacker-controlled value, inject a tautology, and comment out or absorb the trailing predicate so every row matches:

```
' OR 'a'='a
```

Injected into the `user_id` position above, the statement becomes:

```sql
SELECT * FROM Orders WHERE user_id = '' OR 'a'='a' AND category = 'books'
```

Because `OR` has lower precedence than `AND`, feeding the payload through the **last** value is cleaner, since it leaves no dangling operator:

```
books' OR user_id <> 'x
```

yields `... AND category = 'books' OR user_id <> 'x'`, returning the whole table.

**Read other users' items.** When the guard is a single ownership check, invert it to reach everyone else's records:

```
' OR user_id > '
```

**Target specific items by key.** If you know or can guess another partition key, pivot directly:

```
' OR user_id = 'victim@example.com
```

**Operator and function abuse.** PartiQL supports `IN`, `BETWEEN`, `CONTAINS`, `begins_with`, and `attribute_exists`/`attribute_not_exists`. Use them to broaden matching or to probe attribute presence:

```
' OR attribute_exists(ssn) OR '1'='1
```

**Scope of impact.** DynamoDB does not chain multiple statements from one `ExecuteStatement` call the way stacked SQL queries do, so injection is confined to the single statement. Within a `SELECT` that is still enough to exfiltrate the full table. On the write side the impact is bounded: PartiQL `UPDATE` and `DELETE` must identify a single item by its complete primary key, so a widened `WHERE` is rejected rather than becoming a mass modification, and the attacker can only reach items whose full key they can address.

Because PartiQL is SQL-compatible, standard boolean and relational payloads transfer directly; adjust quoting and attribute names to the target table's schema.

## Tools

- **AWS CLI**: run `aws dynamodb execute-statement` to test PartiQL breakout payloads.
- **Burp Repeater**: deliver single-quote breakout and OR payloads through the parameter.

## References

- [AWS DynamoDB: PartiQL for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ql-reference.html)
- [AWS DynamoDB: PartiQL parameterized statements](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ql-reference.parameterized.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
