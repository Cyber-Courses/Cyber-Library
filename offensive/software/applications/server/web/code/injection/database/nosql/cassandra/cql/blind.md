---
title: "Blind inference: boolean extraction from CQL queries"
description: "When a Cassandra query reflects no data, an attacker reads it one fact at a time by crafting predicates whose truth changes whether any rows return."
keywords:
  - blind CQL injection
  - boolean inference
  - Cassandra
  - ALLOW FILTERING
  - data extraction
---

# Blind inference

When an injectable CQL query returns no visible data, only a difference in behavior between a true and a false condition, data is recovered one bit at a time. The attacker supplies a predicate whose truth they want to learn and observes whether the response changes.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Why boolean, not time

CQL has no sleep, benchmark, or deliberately expensive function, so there is no time-based channel to lean on. The only general inference primitive is **boolean**: whether the query returns rows, and how the application reacts to that. The observable signal is usually one of:

- A page that renders differently when at least one row matches versus none.
- A login, lookup, or toggle that succeeds or fails depending on the row count.
- A size or status difference between the two responses.

## The true/false pair

Start from a condition known to be true and one known to be false, and confirm the two produce distinguishable responses. Given a sink that filters on a partition key, a block comment discards the template tail and `ALLOW FILTERING` makes the added predicate acceptable:

```sql
existing' AND role = 'admin' ALLOW FILTERING /*
existing' AND role = 'nonexistent_zzzz' ALLOW FILTERING /*
```

If the first returns rows and the second does not, the channel works and the target has at least one admin row.

## Extracting a value character by character

CQL supports range comparisons and substring tests on text, which is enough to walk a value. `token()` neutralizes the partition-key requirement so the scan reaches the target row:

```sql
' AND token(id) >= token('') AND username = 'admin' AND password > 'm' ALLOW FILTERING /*
```

A binary search over the comparison bound (`>`, `>=`, `<`) narrows each character. Where the stored column is text and supports prefix matching, `LIKE` (on tables with a suitable SASI index) or a sequence of range bounds tightens each position:

```sql
' AND username = 'admin' AND password >= 'ma' AND password < 'mb' ALLOW FILTERING /*
```

Each probe answers one comparison; the response's rows-or-no-rows state is the oracle. Repeat, advancing the known prefix, until the full value is reconstructed.

## Confirming presence of rows and columns

Boolean inference also confirms structural facts without reading them directly. Testing whether a value exists anywhere in a scanned column:

```sql
' AND email = 'target@example.com' ALLOW FILTERING /*
```

A rows-returned response confirms the record exists. Iterating over candidate values enumerates membership, slowly but reliably, over a channel that reflects nothing but presence.

## Automating

Each character typically costs a handful of requests under binary search. Scripting the probe loop, submitting the payload, classifying the response as true or false, and advancing the search, makes extraction of a multi-character secret practical. Error text, where it leaks, gives a faster channel; see [Error-based](error-based.md).

## References

- [Apache Cassandra: SELECT and operators](https://cassandra.apache.org/doc/latest/cassandra/cql/dml.html#select)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
