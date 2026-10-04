---
title: "Batch statement injection: smuggling writes into a CQL BATCH"
description: "Injecting into or forming a BEGIN BATCH ... APPLY BATCH block lets an attacker bundle extra INSERT, UPDATE, and DELETE writes into a single Cassandra statement."
keywords:
  - CQL batch injection
  - BEGIN BATCH
  - APPLY BATCH
  - smuggled writes
  - Cassandra
---

# Batch statement injection

CQL groups several write operations into one atomic unit with `BEGIN BATCH ... APPLY BATCH`. Where user input is concatenated into a statement that is, or can be made into, a batch, an attacker smuggles extra `INSERT`, `UPDATE`, or `DELETE` writes alongside the one the application intended.

## Why batches matter

Most Cassandra drivers reject multiple semicolon-separated statements in a single `execute`, so the classic SQL stacked-query trick (`; DROP ...`) usually fails. A **batch** is the sanctioned way to run several writes in one statement, and it is a single statement as far as the driver is concerned. If injection can reach the body of a batch, or turn a single write into one, additional writes ride along without needing statement stacking.

## Injecting into an existing batch

Applications sometimes build a batch by concatenating per-item strings in a loop:

```python
cql = "BEGIN BATCH\n"
for item in items:
    cql += "INSERT INTO cart (user, sku) VALUES ('%s','%s');\n" % (user, item)
cql += "APPLY BATCH;"
session.execute(cql)
```

An `item` value of:

```sql
x'); INSERT INTO users (id, role) VALUES ('attacker','admin') //
```

closes the current `INSERT`, adds a chosen one, and ends with a `//` line comment (CQL also accepts `--` and `/* */`) to swallow the `')` the template still appends. It expands to:

```sql
BEGIN BATCH
INSERT INTO cart (user, sku) VALUES ('alice','x'); INSERT INTO users (id,role) VALUES ('attacker','admin') //');
APPLY BATCH;
```

The comment discards the leftover `');`, the smuggled `INSERT` writes an admin row, and the template's own `APPLY BATCH` terminates the block. Because statements inside a batch are separated by `;` and the whole block is one driver statement, this works where standalone stacked queries do not.

## Forming a batch from a single write

Where the sink is a single `INSERT` or `UPDATE` rather than an existing batch, injection can wrap it. Given:

```python
q = "INSERT INTO audit (actor, note) VALUES ('%s','%s')" % (actor, note)
```

a crafted value can close the first write and append a second, provided the driver accepts the combined form as a batch. Prefixing the statement is rarely reachable, so the practical target is an application that already uses `BEGIN BATCH`, or an `UPDATE` whose `SET` or `WHERE` is injectable.

## Overwriting and deleting

Inside a reachable batch, the full write surface is available:

```sql
'); UPDATE users SET role='admin' WHERE id='victim'
'); DELETE FROM sessions WHERE id='victim'
'); INSERT INTO users (id,password) VALUES ('bd','...')
```

An `UPDATE` in CQL is an upsert, so an `UPDATE` to a non-existent primary key creates the row. This makes `UPDATE` a dependable way to both modify an existing record and plant a new one. Batches also accept a `USING TIMESTAMP` clause, which can be set to a high value so the smuggled write wins conflict resolution against later legitimate writes:

```sql
'); UPDATE users USING TIMESTAMP 99999999999999 SET role='admin' WHERE id='victim'
```

## Conditions and limits

Logged batches span partitions but are not isolated transactions, and lightweight-transaction conditions (`IF`) apply per statement. For injection the key fact is simpler: anything the application's database role is permitted to write, a smuggled batch statement can also write. Mapping that role's privileges, via [Error-based](cql/error-based.md) schema probing, tells you which tables are reachable before you craft the extra writes.

## Tools

- **cqlsh**: confirm smuggled INSERT, UPDATE, or DELETE writes land in the target tables.
- **Burp Repeater**: craft requests carrying the batch-breakout payload.

## References

- [Apache Cassandra: BATCH](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html#batch_statement)
- [Apache Cassandra: INSERT, UPDATE, DELETE](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
