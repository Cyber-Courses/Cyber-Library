---
title: "MongoDB query-operator injection"
description: "When attacker-controlled objects reach a MongoDB filter, query operators replace the intended comparison, enabling bypass, blind extraction, and denial of service."
keywords:
  - MongoDB operators
  - query operator injection
  - NoSQL injection
  - comparison operators
  - $regex
  - $where
---

# Operator

A MongoDB filter expresses conditions through **query operators**: special keys beginning with `$` that tell the engine how to compare a field. `{ "age": { "$gt": 18 } }` means "age greater than 18". When attacker-controlled data reaches a filter as an object rather than a coerced scalar, the attacker supplies those operators directly, and the comparison the developer intended is replaced by one of their choosing.

The precondition throughout this section is that the application passes an attacker-controlled object into the query without casting it to a string, so the injected `$`-key is parsed as an operator instead of matched as literal data. Request JSON spread into a filter, and query-string or body parsers that build nested objects from bracket notation, are the common routes.

The pages here cover the operator families worth reaching for: comparison operators (`$ne`, `$gt`, `$lt`, `$gte`) for bypass and blind extraction, array operators (`$in`, `$nin`, `$all`, `$elemMatch`) driven from attacker arrays, `$exists` as a field-presence oracle, `$regex` for anchored character-by-character extraction and ReDoS, and `$where`/`mapReduce` for server-side JavaScript where it is enabled.

## Tools

- **NoSQLMap**: automated operator injection and blind extraction.
- **nosqli**: scanner for MongoDB operator injection in web parameters.
- **mongosh**: validate operator filters directly against a database.
- **Burp Suite**: intercept and tamper with query-backed requests.

## References

- [MongoDB: Query and projection operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
