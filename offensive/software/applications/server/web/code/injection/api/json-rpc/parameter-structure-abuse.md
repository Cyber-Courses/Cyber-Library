---
title: "JSON-RPC parameter structure abuse"
description: "When params are accepted as an array or an object inconsistently, missing keys coerce to defaults, alternate code paths fire, or lenient parsers leak routing through errors."
keywords:
  - JSON-RPC params
  - params array
  - params object
  - parameter coercion
  - default values
---

# Parameter structure abuse

JSON-RPC allows `params` to be either a positional array or a named object, and the server binds whichever it receives to the target procedure's arguments. When that binding is lenient, guessing at missing values, accepting both shapes interchangeably, or coercing types, the caller controls which argument values the procedure actually runs with, including ones that select a privileged path or drop a constraint.

## Array versus object binding

A procedure expecting named parameters may also accept a positional array, and the mapping between positions and names is where assumptions break. Sending the other shape than the server was tested with can skip a parameter, reorder trust-relevant ones, or fill an argument the client was not supposed to set:

```json
{"jsonrpc": "2.0", "method": "createUser",
 "params": {"name": "x", "role": "admin"}, "id": 1}
```

If the public form takes `params: ["x"]` and the server binds a named object onto the same procedure, a field like `role` that the positional form never exposed becomes settable. The converse, sending an array where an object was expected, can land values in the wrong arguments.

## Missing keys and defaults

When a required field is omitted and the binder supplies a default rather than rejecting the call, the default may be the insecure case. Dropping an owner, a tenant, or a flag field lets the procedure run with a permissive default:

```json
{"jsonrpc": "2.0", "method": "listInvoices", "params": {}, "id": 1}
```

If `tenantId` defaults to an unscoped or administrative value when absent, the empty params return more than the caller should see.

## Type coercion and extra fields

Lenient binders coerce types (a string where a number was expected, an object where a scalar was) and ignore or absorb extra fields. Supplying an unexpected type can trigger an alternate code path or an error that reveals internal routing, and extra fields can reach an over-permissive binder that maps them onto arguments the schema did not list. Verbose JSON-RPC error objects compound this by returning stack or dispatch detail that maps the parameter handling.

## Confirming the flaw

Send each procedure both as an array and as an object, omit required fields to observe defaulting, supply unexpected types, and add extra named fields. A call that succeeds with a shape or a field the documented form did not allow, or that returns a revealing error, shows the binder is guessing rather than validating against a fixed parameter schema. Strict per-method schema validation of the params shape before dispatch is the control this is probing for.

## References

- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
