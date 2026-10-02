---
title: "WHERE clause injection: breaking out of quoted CQL values"
description: "Closing a quoted CQL value and appending predicates lets an attacker widen a Cassandra filter, subject to the partition-key constraint that WHERE normally enforces."
keywords:
  - CQL WHERE injection
  - quoted value breakout
  - partition key constraint
  - predicate injection
  - Cassandra
---

# WHERE clause injection

The injectable value most often lands inside a quoted literal in a `WHERE` clause. A single quote closes that literal, and whatever follows is parsed as CQL syntax.

## Breaking out of the literal

Given the sink:

```python
q = "SELECT * FROM users WHERE username = '" + name + "'"
```

a value of `' OR username = 'admin` rewrites the statement to:

```sql
SELECT * FROM users WHERE username = '' OR username = 'admin'
```

The trailing `'` from the template closes the literal the attacker opened, leaving valid syntax. CQL accepts `--` and `//` line comments as well as `/* ... */` block comments, so any of them can discard the rest of the template:

```sql
' OR username = 'admin' /*
```

String literals use single quotes; a literal single quote inside a value is escaped by doubling it (`''`), which matters where a filter strips raw quotes but passes the doubled form through to the parser.

## The partition-key constraint

CQL does not behave like SQL here. A table is partitioned by its partition key, and Cassandra normally **rejects** a `WHERE` that does not constrain the full partition key, or that filters on a non-key column, unless the query carries `ALLOW FILTERING`. So this classic breakout:

```sql
SELECT * FROM users WHERE username = '' OR username = 'admin'
```

works only if `username` is the partition key. `OR` is itself restricted in CQL: it is not a general cross-column operator the way it is in SQL, so `' OR 1=1 --`-style payloads usually fail outright. A breakout that targets a non-key column, or that broadens beyond a single partition, generally has to add `ALLOW FILTERING` to be accepted (see [ALLOW FILTERING abuse](allow-filtering-abuse.md)).

## Adding and relaxing predicates

Where the partition key is constrained, you can still widen the result within or across the key using CQL operators that the engine accepts:

```sql
' AND role IN ('admin','superuser') ALLOW FILTERING /*
' AND token(id) > token('') ALLOW FILTERING /*
```

`IN` enumerates several values; `token()` lets you reason over the ring. `CONTAINS` and `CONTAINS KEY` widen collection columns:

```sql
' AND permissions CONTAINS 'admin' ALLOW FILTERING /*
```

For a key column, supplying a different partition value simply pivots to another partition:

```sql
admin' /*
```

turns a lookup for the submitted name into a lookup for `admin`, with the block comment swallowing the template tail.

## Probing the point

Submit a lone `'` and watch for a CQL parse error surfacing in the response (`SyntaxException`, `line 1:...`), a reliable signal the value reaches query text. Compare a benign value against `existing_value' /*` to confirm the comment and breakout parse. Where no rows are reflected and errors are suppressed, fall back to [Blind inference](blind.md); where errors leak, see [Error-based](error-based.md).

## Tools

- **cqlsh**: test quoted-value breakouts and predicate additions directly.
- **Burp Repeater**: craft and resend the breakout payload through the parameter.

## References

- [Apache Cassandra: SELECT and the WHERE clause](https://cassandra.apache.org/doc/latest/cassandra/cql/dml.html#select)
- [Apache Cassandra: Data definition and partition keys](https://cassandra.apache.org/doc/latest/cassandra/cql/ddl.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
