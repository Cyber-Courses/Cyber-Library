---
title: "CQL injection"
description: "Untrusted input concatenated into Cassandra Query Language statements is parsed as syntax, letting an attacker escape quoted values, add predicates, and widen reads with ALLOW FILTERING."
keywords:
  - CQL injection
  - Cassandra Query Language
  - WHERE clause injection
  - ALLOW FILTERING
  - parameter binding
---

# CQL

CQL (Cassandra Query Language) is how applications read and write a Cassandra cluster. It supports **bound parameters**, positional `?` or named `:name`, which the driver sends out-of-band from the statement text. When code instead concatenates user input into the statement string, the input is parsed as CQL and the query becomes injectable.

## Vulnerable pattern

```python
# user_id taken from the request
q = "SELECT * FROM users WHERE id = '" + user_id + "'"
session.execute(q)
```

The safe form binds a value (`WHERE id = %s` with a parameter tuple); the injectable form interpolates it straight into the quoted literal.

## What CQL does and does not give you

CQL reads like SQL but is far more restricted, and payloads that ignore this just raise syntax errors. There is **no `UNION`**, no subqueries, no joins, and no arbitrary `OR` across columns. There is no sleep, benchmark, or heavy-computation function to build time-based inference on. The usable primitives are narrow and specific:

- Close a quoted value and append syntax (see [WHERE clause injection](where-clause-injection.md)).
- Relax or add predicates on columns the query already filters.
- Append `ALLOW FILTERING` to escape the intended partition and scan the whole table (see [ALLOW FILTERING abuse](allow-filtering-abuse.md)).
- Infer data from whether rows return (see [Blind inference](blind.md)) or from error text (see [Error-based](error-based.md)).

A further constraint shapes every `WHERE` payload: Cassandra normally requires the partition-key columns to be constrained, and rejects filters on non-key columns unless `ALLOW FILTERING` is present. That rule is both an obstacle and, deliberately abused, a lever.

Statement separation with `;` depends on the driver. Many client libraries reject multiple statements in one `execute`, so the reliable surface is a single rewritten statement, with `BEGIN BATCH` being the way to bundle several writes into one (see [Batch statement injection](../batch-statement-injection.md)).

This section covers the core `WHERE` mechanics, `ALLOW FILTERING` abuse, and blind and error-based inference.

## Tools

- **cqlsh**: issue and verify CQL payloads against the cluster.
- **Burp Suite**: intercept and tamper with CQL-backed web requests.

## References

- [Apache Cassandra: CQL reference](https://cassandra.apache.org/doc/latest/cassandra/cql/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
