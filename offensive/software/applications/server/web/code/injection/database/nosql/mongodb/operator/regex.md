---
title: "MongoDB $regex for anchored extraction and ReDoS"
description: "The $regex operator anchors patterns to recover string fields character by character and, with catastrophic patterns, drives regex denial of service."
keywords:
  - $regex
  - MongoDB injection
  - anchored prefix extraction
  - blind extraction
  - ReDoS
  - denial of service
---

# Regex

`$regex` matches a string field against a regular expression. Injected into a filter it is the sharpest tool for reading string data through a boolean oracle, and, with a deliberately expensive pattern, a lever for denial of service.

The precondition is that the field reaches the filter as an attacker-controlled object, so `{"$regex": "..."}` is parsed as an operator.

## Anchored prefix extraction

The core technique anchors a pattern at the start of the value with `^` and extends it one character at a time. Each request answers "does the field start with this prefix", and the oracle (a login success, a redirect, a row count, a response-length difference) carries the answer:

```json
{"username": "admin", "apiKey": {"$regex": "^a"}}
{"username": "admin", "apiKey": {"$regex": "^ab"}}
{"username": "admin", "apiKey": {"$regex": "^abc"}}
```

The extraction loop: hold the confirmed prefix, append each candidate character in turn, submit, and read the oracle; the character that keeps it true is correct, so append it and move on. Through a nested-object parser the same probe is:

```
username=admin&apiKey[$regex]=^abc
```

Narrow the character set to speed the walk. Case-insensitive matching with the `$options` flag halves the alphabet when case is not needed, and a character class tests a whole group per request:

```json
{"username": "admin", "apiKey": {"$regex": "^abc", "$options": "i"}}
{"username": "admin", "apiKey": {"$regex": "^abc[0-9a-f]"}}
```

A hex class like `[0-9a-f]` is ideal against tokens and hashes, confirming the next character's family in one request before pinning the exact value. Learn the length first with an anchored length probe, matching only values of at least N characters:

```json
{"username": "admin", "apiKey": {"$regex": "^.{32}"}}
```

Escape regex metacharacters (`.`, `*`, `+`, `?`, `(`, `)`, `[`, `]`, `{`, `}`, `^`, `$`, `|`, `\`) in candidate characters so each is matched literally rather than as syntax. The result is full recovery of any string field reachable by the filter, one request per character or character-class tested. This is the extraction engine behind [Comparison operators](comparison-operators.md) and [Array operators](array-operators.md).

## ReDoS denial of service

Where the application lets attacker input become the pattern (rather than the subject) of a regex, a catastrophic-backtracking expression stalls the matching engine against long inputs. Nested quantifiers over an overlapping class are the classic form:

```json
{"field": {"$regex": "^(a+)+$"}}
{"field": {"$regex": "(a|a)*$"}}
{"field": {"$regex": "(.*a){50}"}}
```

Matched against a sufficiently long stored (or attacker-supplied) value that almost satisfies the pattern, these expressions explode into exponential backtracking, pinning CPU on the query path and starving other requests. A single such query, or a handful fired in parallel, is enough to degrade a service. The same pattern class is effective wherever user input reaches a regex, so an endpoint that feeds the pattern into `$regex` is a direct denial-of-service primitive as well as an extraction one.

## Tools

- **Burp Intruder**: automate the anchored per-character $regex extraction walk.
- **nosqli**: detect $regex injection points in parameters.
- **NoSQLMap**: automate regex-based blind extraction.

## References

- [MongoDB: $regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
