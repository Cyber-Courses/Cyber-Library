---
title: "ALLOW FILTERING abuse: forcing full-cluster CQL scans"
description: "Appending ALLOW FILTERING to an injected CQL query removes the partition-key restriction, letting an attacker scan every node and read rows beyond the intended partition."
keywords:
  - ALLOW FILTERING
  - full table scan
  - partition key bypass
  - CQL injection
  - Cassandra data exposure
---

# ALLOW FILTERING abuse

Cassandra protects itself by rejecting any query whose `WHERE` does not constrain the partition key, or that filters on a non-key column, because such a query cannot be served from a single partition and must instead touch every node. The escape hatch is the `ALLOW FILTERING` keyword, which tells the coordinator to run that expensive scan anyway. Injected into a string-built query, it converts a narrow partition lookup into a cluster-wide read.

## Why it matters for injection

An application usually scopes a query to one partition the current user is allowed to see:

```python
q = "SELECT * FROM messages WHERE account_id = '" + acct + "' AND box = '" + box + "'"
```

Both predicates sit on key columns, so the query is cheap and tightly scoped. Without `ALLOW FILTERING`, an injected predicate on a **non-key** column is simply refused by the engine, so the attacker's broadened filter never runs. Adding the keyword removes that refusal, and the broadened filter is evaluated against rows from every partition on every node.

## Reading beyond the partition

Given the sink above, a `box` value of:

```sql
inbox' AND priority = 'high' ALLOW FILTERING /*
```

produces:

```sql
SELECT * FROM messages WHERE account_id = '...' AND box = 'inbox'
AND priority = 'high' ALLOW FILTERING
```

The added predicate is an equality on a non-key scalar column, which CQL refuses without `ALLOW FILTERING`. (`CONTAINS` is not an option here: it tests membership in a collection column, not a substring of a `text` column. Substring matching needs `LIKE` against a SASI-indexed column, covered in [blind extraction](blind.md).)

Scanning beyond a single partition requires a sink where no fixed partition-key equality precedes the injection, because `ALLOW FILTERING` does not remove an existing `account_id = '...'` predicate (that equality still pins the query to one partition). The case that works is a query filtering only on the injected non-key column, such as an admin or search endpoint `SELECT * FROM messages WHERE status = '<inj>'`, where a `token()` range then sweeps the whole partitioner ring:

```sql
x' AND token(account_id) >= -9223372036854775808 ALLOW FILTERING /*
```

The bound is the Murmur3 partitioner's minimum token, so the range covers the entire ring; `ALLOW FILTERING` is what actually permits the cross-partition scan. Non-key equality and collection-membership predicates then pick out the rows of interest:

```sql
x' AND role = 'admin' ALLOW FILTERING /*
x' AND permissions CONTAINS 'billing' ALLOW FILTERING /*
```

## Combining with IN and ranges

`ALLOW FILTERING` pairs with the operators CQL does allow, letting a single statement sweep a wide value space:

```sql
' AND status IN ('active','locked','pending') ALLOW FILTERING /*
' AND age >= 0 ALLOW FILTERING /*
```

The second payload, with an always-true range on a non-key numeric column, returns every row the engine can reach, the closest CQL equivalent of a `1=1` dump, and depends entirely on `ALLOW FILTERING` to be accepted.

## Placement

The keyword must come after the `WHERE` predicates and before any `LIMIT` the engine applies. A trailing `/*` block comment discards the remainder of the original statement so a stray `LIMIT` or `ALLOW FILTERING` from the template does not break parsing:

```sql
' AND role = 'admin' ALLOW FILTERING /*
```

Where output is not reflected, the same full-scan predicates drive row-count inference in [Blind inference](blind.md).

## Tools

- **cqlsh**: validate ALLOW FILTERING and token-range payloads directly against the cluster.
- **Burp Repeater**: deliver the broadened predicate through the web parameter.

## References

- [Apache Cassandra: ALLOW FILTERING](https://cassandra.apache.org/doc/latest/cassandra/cql/dml.html#allow-filtering)
- [Apache Cassandra: SELECT](https://cassandra.apache.org/doc/latest/cassandra/cql/dml.html#select)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
