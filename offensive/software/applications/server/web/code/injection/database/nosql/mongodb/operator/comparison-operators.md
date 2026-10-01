---
title: "MongoDB comparison-operator injection for bypass and blind extraction"
description: "Injecting $ne, $gt, $lt, and $gte replaces the intended comparison to bypass filters, and pairs with $regex to extract data one character at a time blindly."
keywords:
  - comparison operators
  - $ne
  - $gt
  - $lt
  - blind NoSQL injection
  - MongoDB data extraction
---

# Comparison operators

The comparison operators (`$ne`, `$gt`, `$lt`, `$gte`, `$lte`, `$eq`) are the workhorses of MongoDB injection. Injected in place of a scalar, they rewrite the comparison a filter performs, which bypasses checks and, through a true/false oracle, extracts data without any rows being reflected.

The precondition is that the field reaches the filter as an attacker-controlled object, not a string. A value cast with `String(input)` is matched literally and none of the following applies.

## Bypassing a filter

`$ne` (not equal) and `$gt` (greater than) against a trivial bound make a condition match every document that has the field:

```json
{"password": {"$ne": null}}
{"password": {"$gt": ""}}
{"active": {"$ne": false}}
```

`{"$ne": null}` matches any non-null value; `{"$gt": ""}` matches any string greater than the empty string, which is every non-empty string. Delivered through a nested-object parser the same payloads are:

```
password[$ne]=
password[$gt]=
active[$ne]=false
```

Used on a login filter this is the [authentication bypass](../authentication-bypass.md); used on a listing or lookup endpoint it widens results past the intended row.

## A boolean oracle for blind extraction

When the response does not reflect data but does differ between a matching and non-matching query (a row count, a redirect, a status code, a response length), comparison operators build a yes/no oracle. Numeric and date fields fall to binary search with `$gt`/`$lt`:

```json
{"username": "admin", "age": {"$gt": 30}}
{"username": "admin", "age": {"$lt": 40}}
{"username": "admin", "age": {"$gte": 35}}
```

Each request answers one comparison; halve the range on each step until the exact value of a numeric field (an age, a balance, a timestamp, an auto-increment-style counter) is pinned down.

## Pairing with $regex for string fields

Comparison operators order values but cannot read a string's contents, so string extraction pairs the same oracle with `$regex`. Anchor a pattern to a known prefix and extend it character by character, keeping each character that keeps the oracle true:

```json
{"username": "admin", "secret": {"$regex": "^s"}}
{"username": "admin", "secret": {"$regex": "^se"}}
{"username": "admin", "secret": {"$regex": "^sec"}}
```

A typical loop: fix the known prefix, try each candidate character appended to the anchor, submit the request, and read the oracle. A true result means the character is correct; move to the next position. Combine with `$gt`/`$lt` on length-bearing fields or with `$regex` length probes (`^.{10}` matches values of at least ten characters) to learn how far to extract.

```
username=admin&secret[$regex]=^sec
```

Escape regex metacharacters (`.`, `*`, `+`, `?`, `(`, `)`, `[`, `]`, `\`, `^`, `$`) in candidate characters so each matches literally. The result is full recovery of string fields (password hashes, tokens, recovery answers) from a single boolean signal, one request per character tested. See [Regex](regex.md) for anchoring detail and [Exists](exists.md) for discovering which fields to target.

## References

- [MongoDB: Query comparison operators](https://www.mongodb.com/docs/manual/reference/operator/query-comparison/)
- [MongoDB: $regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
