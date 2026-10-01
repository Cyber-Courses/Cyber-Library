---
title: "Elasticsearch and OpenSearch injection"
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

