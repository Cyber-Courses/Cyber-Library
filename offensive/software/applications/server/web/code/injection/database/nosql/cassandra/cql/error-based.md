---
title: "Error-based: leaking Cassandra schema through CQL errors"
order: 2
description: "CQL error messages name tables, columns, and expected types, letting an attacker map a Cassandra schema by deliberately provoking type mismatches and unknown-identifier errors."
keywords:
  - error-based CQL injection
  - Cassandra schema disclosure
  - type mismatch
  - unknown identifier
  - InvalidRequest
---

# Error-based

When CQL errors propagate back to the response, they carry schema and type detail that the application never meant to expose. An attacker who can trigger errors on demand reads structure directly from the message text instead of inferring it.

## What CQL errors reveal

Cassandra reports precise, structured errors. The useful ones for mapping a schema:

- **`SyntaxException`** confirms the input reaches query text and often echoes the offending token and line position (`line 1:42 ...`), pinpointing where the value lands.
- **`InvalidRequest` / unknown identifier** names columns and tables. Referencing a column that does not exist returns `Undefined column name <x>`, so probing candidate names enumerates the real ones.
- **Type-mismatch** errors state the expected CQL type. Comparing a text column against an integer, or vice versa, returns a message naming the column's declared type.

## Provoking a type mismatch

If a value is compared against a column, forcing an incompatible comparison makes the engine report the column's type. Given a text sink, injecting an integer comparison:

```sql
' AND age = 'not_a_number' ALLOW FILTERING /*
```

against an `int` column yields a message along the lines of `Expected 4 or 0 byte int` or `Unable to make int from 'not_a_number'`, confirming `age` is an integer. Reversing the mismatch confirms a text column:

```sql
' AND username = 12345 ALLOW FILTERING /*
```

returning `Expected 8 or 0 byte long` or an invalid-type message that names the declared type.

## Enumerating columns and tables

Unknown-identifier errors turn guesses into confirmations. Each candidate name either parses (the column exists) or returns `Undefined column name`:

```sql
' AND password = 'x' ALLOW FILTERING /*
' AND password_hash = 'x' ALLOW FILTERING /*
' AND is_admin = true ALLOW FILTERING /*
```

A name that does not error is a real column; one that returns the undefined-column message is not. The same technique against table names, where the sink allows reaching the `FROM` target, confirms table existence.

## Reading the system keyspace

Where the injection can reach a full `SELECT` target rather than only a `WHERE` value, the `system_schema` keyspace holds the catalog, and error text while probing it still leaks detail even when rows are not reflected:

```sql
SELECT column_name, type FROM system_schema.columns WHERE keyspace_name = 'app' AND table_name = 'users'
```

`system_schema.tables`, `system_schema.columns`, and `system_schema.keyspaces` describe every keyspace, table, column, and type. Combined with the error oracle above, they let an attacker reconstruct the schema before moving to data extraction via [Blind inference](blind.md).

## Keeping errors visible

Error-based extraction depends on messages surviving to the response. Submitting a lone `'` to confirm a `SyntaxException` is echoed is the first check; if it is swallowed, the channel is boolean-only. Where messages are reflected, they are the fastest way to map structure before data recovery begins.

## Tools

- **cqlsh**: reproduce type-mismatch and unknown-identifier errors to read schema.
- **Burp Repeater**: submit error-provoking payloads and inspect returned messages.

## References

- [Apache Cassandra: system_schema tables](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html)
- [Apache Cassandra: CQL data types](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/types.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
