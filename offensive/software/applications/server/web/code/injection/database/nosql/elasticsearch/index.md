---
title: "Elasticsearch and OpenSearch injection"
order: 5
description: "User input reaching Lucene query_string syntax, the JSON Query DSL body, or Painless scripts reopens injection on Elasticsearch and OpenSearch clusters."
keywords:
  - Elasticsearch injection
  - OpenSearch injection
  - Lucene query_string
  - Query DSL
  - Painless scripting
---

# Elasticsearch

Elasticsearch and its fork OpenSearch are schema-flexible document stores queried over HTTP, and their query surface is unusually wide. Three distinct layers accept untrusted input and reopen injection. The **Lucene `query_string` / `simple_query_string`** syntax lets a search box carry boolean operators, field selectors, wildcards, ranges, and reserved tokens, all interpreted rather than escaped. The **JSON Query DSL** request body is frequently assembled by string concatenation or by spreading request JSON into a `bool` clause, letting an attacker add or replace clauses. And **Painless scripting**, exposed through `script_fields`, `script` queries, sorts, and updates, runs attacker-influenced source when input is interpolated into script text.

Because the same APIs back both engines, payloads here apply to Elasticsearch and OpenSearch alike. The child pages split the surface by sink: query-string syntax, Query DSL body, Painless script injection, and boolean-based blind extraction.

## Pages

- **[Blind boolean extraction](blind-boolean.md)**: When only hit-or-no-hit is observable, crafted field conditions on query_string, Query DSL, or script queries let an attacker infer and extract document valu...
- **[Query DSL injection](query-dsl-injection.md)**: When user input is concatenated into the JSON Query DSL or spread into a bool clause, an attacker injects should/must_not clauses or replaces the query to re...
- **[Query string injection](query-string-injection.md)**: When a search term flows unescaped into query_string or simple_query_string, Lucene operators let an attacker retarget fields, flip boolean logic, and widen...
- **[Script injection](script-injection.md)**: When user input is interpolated into Painless script source in script_fields, script queries, sorts, or updates, an attacker reads arbitrary document data an...

## Tools

- **curl**: send raw _search and script requests to the cluster HTTP API.
- **Burp Suite**: intercept and tamper with search-backed web requests.

## References

- [Elasticsearch: Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)

