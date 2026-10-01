---
title: "JSON-RPC procedure name bypass"
description: "When the method string is mapped to a server procedure through unsafe reflection, case tricks, delimiter smuggling, or prefix stripping, a caller reaches internal or admin methods the allowlist thought it excluded."
keywords:
  - JSON-RPC method
  - reflection dispatch
  - procedure allowlist
  - method name bypass
  - internal method
---

# Procedure name bypass

A JSON-RPC server maps the request's `method` string to a function. How it does that mapping decides what a caller can reach. An explicit table from allowed names to specific functions is safe; a dynamic lookup that resolves the string against an object, a module, or a naming convention is not, because the caller controls the string and can steer it to functions the designer never meant to expose.

## Reflection-based dispatch

The dangerous pattern resolves the method name directly against a host object, for example `getattr(handler, method)` in Python, a reflective method lookup in Java, or indexing a module by name. Any attribute or function reachable that way becomes callable:

```json
{"jsonrpc": "2.0", "method": "_admin_reset", "params": [], "id": 1}
```

Where dispatch is `getattr(self, request.method)`, a method name beginning with an underscore, or any internal helper on the handler object, is invoked even though only the public methods were documented. Names like `__class__`, `__init__`, or framework internals may also resolve, reaching well beyond the intended API.

## Name normalization gaps

Even with a name filter, the check and the dispatch can normalize the name differently. A case-sensitive denylist that blocks `admin.deleteUser` does not match a differently cased spelling, while a case-insensitive dispatcher still resolves that spelling to the blocked function, so the variant slips past a denylist (and past any coarse check that normalizes differently than the dispatcher):

```json
{"jsonrpc": "2.0", "method": "Admin.DeleteUser", "params": {"id": 104}, "id": 1}
```

Delimiter and namespace tricks exploit the same split: where names are `namespace.method` and the allowlist matches only the prefix, or strips a prefix before dispatch, an injected separator or a crafted prefix reaches a different method than the one authorized. Leading or trailing whitespace, alternate separators (`.`, `/`, `:`), and encoded characters are each worth testing against the gap between how the name is validated and how it is resolved.

## Confirming the flaw

Enumerate method names by probing: common admin and internal prefixes (`admin`, `internal`, `debug`, `__`), reflection-reachable attributes, and case and separator variants of known methods. A method that executes, or that returns a different error (an argument error rather than method-not-found) when it should be blocked, reveals that the name resolved to a real function past the name filter. The safe design, an explicit string-to-function map with no dynamic lookup, is the thing whose absence this probes.

## References

- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
