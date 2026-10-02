---
title: "JSON-RPC abuse: method and parameter dispatch"
description: "JSON-RPC maps a method name and params to a server procedure. The attack surface is how the method string resolves to a function, how params shape is handled, and how batch arrays are authorized."
keywords:
  - JSON-RPC injection
  - method dispatch
  - params structure
  - batch request
  - procedure allowlist
---

# JSON-RPC

JSON-RPC is minimal: a request names a `method`, carries `params`, and the server maps the name to a procedure and calls it with those params. That thinness pushes all the security decisions into application code, how the method string becomes a function, how params are coerced into arguments, and how a batch of calls is authorized, and each is a place where a lenient implementation lets a caller reach more than intended.

## The three surfaces

- **[Procedure name bypass](procedure-name-bypass.md)**: the `method` string mapped to a procedure through unsafe reflection, case tricks, delimiter smuggling, or prefix stripping, reaching internal or admin methods the allowlist thought it excluded.
- **[Parameter structure abuse](parameter-structure-abuse.md)**: `params` accepted as an array or an object inconsistently, so missing keys coerce to defaults, alternate code paths fire, or lenient parsers leak routing in errors.
- **[Batch request abuse](batch-request-abuse.md)**: a batch array where one entry fails authorization open while another performs a sensitive method, exploiting per-item handling and error policy.

## Why minimal shifts the risk to code

There is no framework of routes, verbs, and content negotiation to lean on; a JSON-RPC server is a dispatch table and a parameter binder written by the application. If that table is built with dynamic attribute lookup instead of an explicit map, the method name reaches more functions than intended. If the binder guesses at params shape instead of validating it, the caller picks the code path. The test is to probe how a method name resolves and how params are parsed, before assuming either is constrained.

## Tools

- **Burp Suite**: intercepting and replaying JSON-RPC method and params payloads.
- **curl**: scripting method-name and params-shape probes against the endpoint.

## References

- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
