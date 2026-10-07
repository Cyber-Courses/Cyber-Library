---
title: "MongoDB $exists as a boolean oracle for field inference"
order: 1
description: "The $exists operator tests whether a field is present, giving a true/false oracle to map document schema and infer data when no rows are reflected."
keywords:
  - $exists
  - MongoDB injection
  - boolean oracle
  - field enumeration
  - schema inference
  - blind injection
---

# Exists

`$exists` tests whether a field is present in a document, returning a pure boolean. Injected into a filter it becomes an oracle for the shape of the data: which fields exist, on which documents, without any value ever being reflected.

The precondition is that the attacker-controlled object reaches the filter untyped, so `{"$exists": true}` is parsed as an operator rather than matched as data.

## Field-presence oracle

Pair `$exists` with a known selector and watch whether the query still matches. The difference between `true` and `false` tells you whether the named field is present on the targeted document:

```json
{"username": "admin", "password": {"$exists": true}}
{"username": "admin", "mfaSecret": {"$exists": true}}
{"username": "admin", "resetToken": {"$exists": true}}
```

If the first matches and the second does not, `admin` has a `password` field but no `mfaSecret`. This maps the document schema one field at a time: probe candidate names (`password`, `passwordHash`, `apiKey`, `ssn`, `isAdmin`, `resetToken`, `stripeCustomerId`) and keep the ones that return true. Through a nested parser:

```
username=admin&resetToken[$exists]=true
```

## Inference and branching

Beyond schema mapping, `$exists` is a clean control for building inference chains. Because it returns a dependable true/false, it anchors the "is there anything to extract here" step before switching to `$regex` or comparison operators for the value itself:

```json
{"email": "victim@example.com", "resetToken": {"$exists": true}}
```

A true result says a live reset token exists for that account, a signal worth acting on before extracting it character by character with [Regex](regex.md). `$exists` also constructs forced-true and forced-false branches for calibrating any boolean oracle: `{"_id": {"$exists": true}}` matches every document (a guaranteed-true baseline), while `{"__nope__": {"$exists": true}}` against a field that cannot exist gives a guaranteed-false baseline. Comparing a probe against these two baselines distinguishes a real negative from an error or empty result.

Combined with the operators in [Comparison operators](comparison-operators.md) and [Array operators](array-operators.md), `$exists` first decides where data lives, then those operators read it.

## Tools

- **Burp Repeater**: submit $exists probes to map which fields are present.
- **nosqli**: scan parameters for operator-object injection.
- **NoSQLMap**: automate field-presence enumeration.

## References

- [MongoDB: $exists](https://www.mongodb.com/docs/manual/reference/operator/query/exists/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
