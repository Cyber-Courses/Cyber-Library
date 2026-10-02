---
title: "Property pollution: prototype pollution and deep-merge abuse in JavaScript stacks"
description: "Injecting __proto__ or constructor.prototype through recursive merge, clone, or path-set helpers so attacker-chosen properties appear on objects the server trusts, bypassing property-level authorization."
keywords:
  - prototype pollution
  - __proto__
  - deep merge
  - BOPLA
---

# Property pollution

## Context

In JavaScript and Node, every object inherits from `Object.prototype`. When server code merges an attacker-controlled JSON body into an object using a recursive deep-merge, clone, or path-set helper that does not guard the `__proto__`, `constructor`, and `prototype` keys, the attacker writes onto that shared prototype. Afterwards every object in the realm appears to carry the injected property, so an authorization check that reads a field such as `user.isAdmin` or `options.role` from a freshly built object sees the attacker's value even though it was never set on that object. This is a property-level authorization bypass: you set a property you were never permitted to set, on objects you do not own.

## Theory

The sink is any routine that walks attacker-chosen keys into a target: hand-rolled deep merge, a recursive `Object.assign`, lodash `merge` / `set` / `defaultsDeep` (historically vulnerable), jQuery `$.extend(true, ...)`, and query-string parsers that build nested objects from `a[b][c]=d`. The payload uses a key the walker treats as an ordinary property name but the engine treats as the prototype pointer. The impact gadget is any property the application reads from an object it just built without explicitly setting it: role and entitlement flags, feature toggles, validation switches, or template and config options (which can push prototype pollution further into RCE or XSS).

## Practice

### Pollute via a merged JSON body

- Where an endpoint merges the request body into a user or options object, send a body carrying `__proto__`:

```json
{ "__proto__": { "isAdmin": true } }
```

If a later response, action, or authorization decision then treats the account as privileged, the deep-merge sink polluted the prototype. When the literal `__proto__` key is stripped, reach the same prototype through the constructor:

```json
{ "constructor": { "prototype": { "isAdmin": true } } }
```

### Pollute via nested query parameters

- Parsers that build nested objects from bracket notation accept the key in the URL or form body:

```
POST /api/profile?__proto__[role]=admin
```

Confirm by reading back an object the server constructs after the request and checking for the injected property, or by observing the privilege change directly.

This is distinct from HTTP parameter pollution (which duplicate parameter wins), covered under [parameter pollution](../../injection/http/parameter-pollution.md); here the payload mutates the object prototype rather than exploiting duplicate-key precedence.

## Tools

- **Burp Suite**
- **curl**
- Prototype-pollution gadget scanners (server and client)
