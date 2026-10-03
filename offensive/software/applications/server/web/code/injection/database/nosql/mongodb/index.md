---
title: "MongoDB injection"
description: "Operator and syntax injection against MongoDB, where request JSON reaches a query unsanitized and attacker-controlled objects are interpreted as query operators."
keywords:
  - MongoDB injection
  - NoSQL injection
  - operator injection
  - query operator
  - type juggling
  - authentication bypass
---

# MongoDB

MongoDB stores documents and is queried with BSON/JSON query objects rather than a text query language, so the classic injection primitive is not a quote breakout but an **operator**. A MongoDB filter is a structure like `{ "username": "admin", "password": "secret" }`, where a bare value means equality. When a key's value is itself an object such as `{ "$ne": null }`, that object is a query operator (`$ne`, `$gt`, `$regex`, `$where`, and others). Injection arises when attacker-controlled data reaches the filter as a structure instead of a coerced scalar: a field the application expects to be a string arrives as an operator object, and the comparison the developer intended is replaced by one the attacker chose.

The decisive precondition is that the app passes attacker-controlled objects into the query without coercing them to strings. This reaches the query when request JSON is spread straight into a filter, or when a query-string or form body parser (qs, body-parser, PHP) turns bracketed keys like `password[$ne]=` into nested objects. This subtree covers authentication bypass, JSON type-juggling, and the individual query operators abused for bypass, blind extraction, and denial of service.

## Subtopics

- **[Operator](operator/index.md)**: When attacker-controlled objects reach a MongoDB filter, query operators replace the intended comparison, enabling bypass, blind extraction, and denial of se...

## Pages

- **[Authentication bypass](authentication-bypass.md)**: Replacing a password string with an operator object such as {\"$ne\":null} turns a login lookup into an always-true filter, logging in without credentials.
- **[JSON injection](json.md)**: A value expected as a string arriving as an object or array turns MongoDB comparisons into operator injection; duplicate keys and parser-built nesting widen...

## Tools

- **NoSQLMap**: automated MongoDB operator injection and enumeration.
- **nosqli**: scanner for MongoDB operator injection in web parameters.
- **mongosh**: the MongoDB shell for validating filters against a target database.
- **Burp Suite**: intercept and tamper with query-backed requests.

## References

- [MongoDB: Query and projection operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
