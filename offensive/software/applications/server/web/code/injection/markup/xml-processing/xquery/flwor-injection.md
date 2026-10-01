---
title: "FLWOR injection: rewriting XQuery iteration and predicates"
description: "Untrusted input inside for/let/where/order by clauses changes which nodes are iterated and which predicates hold, enabling literal breakout and tautologies."
keywords:
  - FLWOR injection
  - XQuery injection
  - where clause breakout
  - XPath predicate injection
  - or 1=1 XQuery
---

# FLWOR injection

A FLWOR expression (`for`, `let`, `where`, `order by`, `return`) is XQuery's iteration construct, the direct analogue of a SQL `SELECT`. When application code builds any of these clauses by concatenating request input, the attacker controls which sequence is bound, which predicate filters it, and how results are ordered. The classic vulnerable pattern puts the input inside a string comparison:

```xquery
for $x in //user
where $x/name = '" + name + "'
return $x/profile
```

With `name` taken verbatim from the request, the single quote is the boundary to escape.

## String-literal breakout

Closing the quote and appending a predicate turns the filter into a tautology. Supplying this value for `name`:

```
' or '1'='1
```

produces `where $x/name = '' or '1'='1'`, which is true for every `$x`, so the loop returns every user's profile. The same works against a predicate built inline:

```xquery
//user[name='" + name + "']
```

with the payload `x'] | //user | a[''='` closing the predicate, unioning the full `//user` set with `|`, and reopening a harmless predicate so the expression stays well-formed.

## Widening the iterated sequence

Where the input sits in the `for` binding rather than the `where`, the attacker redirects iteration itself. Starting from:

```xquery
for $x in //user[role='" + role + "']
return $x/name
```

the value `x'] return $x/* for $x in //user[role='admin` reshapes the `return` to emit every child element of each node and re-binds the loop, pulling fields the query never intended to expose (password hashes, tokens, internal flags).

## Cross-document reach

Because predicates are just boolean expressions, an injected one can pivot to other parts of the tree with an absolute path. The fragment `' or //secret/text() or '` forces evaluation of `//secret` anywhere in the document, and the truthiness of that node set leaks through the result set or through timing.

## Blind boolean inference

When rows are not reflected, a controllable predicate still leaks one bit per request. The payload `' or substring((//user[1]/password),1,1)='a` returns the full set only when the guessed character matches, so a scripted binary or linear search over `substring()` and `string-length()` reconstructs any value character by character. `fn:string-to-codepoints()` narrows each position with a numeric comparison instead of an alphabet walk.

## References

- [W3C XQuery 3.1: FLWOR Expressions](https://www.w3.org/TR/xquery-31/#id-flwor-expressions)
- [PayloadsAllTheThings: XPath Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
