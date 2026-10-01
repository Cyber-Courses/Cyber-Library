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
