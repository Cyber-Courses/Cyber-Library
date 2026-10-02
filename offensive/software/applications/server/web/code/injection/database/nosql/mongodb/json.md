---
title: "JSON injection and type juggling in MongoDB queries"
description: "A value expected as a string arriving as an object or array turns MongoDB comparisons into operator injection; duplicate keys and parser-built nesting widen the sink."
keywords:
  - JSON injection
  - type juggling
  - MongoDB
  - parameter pollution
  - duplicate keys
  - operator injection
---

# JSON injection

MongoDB operator injection is, at root, a **type-confusion** bug. The developer expects a scalar (a string or number) at a given key and compares against it; the attacker supplies a different JSON type (an object or an array) and the meaning of the query changes. Understanding how a value's type is chosen, and how parsers build that type, is what makes the operator payloads elsewhere in this subtree reliable.

## A string expected, an object supplied

A bare value in a filter means equality. The moment the value is an object, MongoDB reads its keys as operators:

```json
{"user": "alice"}
{"user": {"$ne": "alice"}}
```

The first matches the document for `alice`. The second matches every document whose `user` is not `alice`. The application wrote `{"user": <input>}` intending equality; by controlling the JSON type of `<input>` the attacker substitutes an operator. Any endpoint that spreads request JSON into a query is exposed:

```javascript
// filter expected to be { status: "active" }
const docs = await col.find({ status: req.query.status }).toArray();
// attacker sends ?status[$ne]=active  ->  { status: { $ne: "active" } }
```

The precondition for every variant below is the same: the value reaches the query as the parsed object or array, with no cast to string. A `String(input)` or schema validation that rejects non-strings closes the type hole.

## An array supplied where a scalar is read

Supplying an array is a second type-juggling primitive. Some operators and some application code behave differently, or crash informatively, when handed an array, and an array is also the payload shape for the array operators:

```json
{"role": {"$in": ["admin", "superuser", "root"]}}
{"id": {"$in": [1, 2, 3, 4, 5]}}
```

An array reaching a field compared with `$in`/`$nin` lets the attacker enumerate or widen matches in a single request. See [Array operators](operator/array-operators.md).

## How query-string and body parsers build nested objects

Attackers rarely control raw JSON on form endpoints; the parser builds the object for them. Parsers that honor bracket notation (Express `qs` and `body-parser`, PHP's native `$_GET`/`$_POST`) convert bracketed keys into nested structures:

```
status[$ne]=active        ->  { status: { $ne: "active" } }
tags[$in][]=a&tags[$in][]=b  ->  { tags: { $in: ["a", "b"] } }
id[$gt]=0                 ->  { id: { $gt: "0" } }
```

This is why a payload delivered as `application/x-www-form-urlencoded` or in a GET query string reaches the query as an operator object identical to the JSON form. The injection does not require a JSON content type, only a parser that produces nesting and code that forwards the result into a filter.

## Duplicate keys and parameter pollution

When the same key appears twice, the chosen value depends on the parser, and the parser the framework uses may differ from the one a WAF or validator inspected. HTTP parameter pollution exploits that gap:

```
username=admin&username[$ne]=x
password=guess&password[$ne]=guess
```

One layer may read the first (string) occurrence and see a benign request; the layer that builds the query may keep the last occurrence (an operator object), so the filter that executes is the malicious one. The same divergence appears in JSON bodies with duplicate keys:

```json
{"password": "guess", "password": {"$ne": null}}
```

Most JSON parsers keep the last value, yielding the operator object, while a scanner that stopped at the first key is bypassed. Probe which occurrence wins by sending a known-good value paired with a known-bad one and observing whether the request authenticates or errors.

## Confirming the type sink

Send the same field three ways and compare responses: a plain string (baseline), an always-true operator (`{"$ne": null}` or `{"$gt": ""}`), and a malformed operator (`{"$ne":` left open, or `{"$foo": 1}`). A behavior change between the string and the operator, or a BSON/cast error from the malformed form, confirms the value lands in the query untyped and the operators in the rest of this subtree apply.

## Tools

- **Burp Repeater**: submit the field as string, operator object, and array to confirm the type sink.
- **nosqli**: scan parameters for type-juggling injection points.
- **NoSQLMap**: automate operator and array payload delivery.

## References

- [MongoDB: Query and projection operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
