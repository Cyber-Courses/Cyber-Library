---
title: "JSON-RPC batch request abuse"
description: "A batch array where one entry fails authorization open while another performs a sensitive method exploits per-item handling and error policy to smuggle privileged calls past per-request controls."
keywords:
  - JSON-RPC batch
  - batch authorization
  - multi-call
  - per-item auth
  - atomic failure
---

# Batch request abuse

JSON-RPC lets a client send an array of requests in a single call, and the server processes each and returns an array of responses. When controls that should apply to every call, authorization, rate limiting, logging, are implemented once for the HTTP request instead of once per array entry, the batch becomes a wrapper that carries privileged or repeated calls under a single authorized envelope.

## Per-request controls applied once

A rate limit or an attempt counter that increments per HTTP request sees one request no matter how many entries the batch holds. Packing many sensitive calls into one batch runs them all under a single tick of the counter:

```json
[
  {"jsonrpc":"2.0","method":"redeemCoupon","params":{"code":"A1"},"id":1},
  {"jsonrpc":"2.0","method":"redeemCoupon","params":{"code":"A2"},"id":2},
  {"jsonrpc":"2.0","method":"redeemCoupon","params":{"code":"A3"},"id":3}
]
```

This is the JSON-RPC form of the amplification that aliasing provides in GraphQL: a throttled operation runs far more times than the per-request limit intended.

## Authorization that fails open mid-batch

When authorization is checked for the batch as a whole, or when a failing entry does not stop the others, an attacker pairs an allowed call with a restricted one. If the server authorizes on the first entry, or continues past an unauthorized entry and still executes later ones, the sensitive call runs:

```json
[
  {"jsonrpc":"2.0","method":"getPublicStatus","params":[],"id":1},
  {"jsonrpc":"2.0","method":"admin.deleteUser","params":{"id":104},"id":2}
]
```

A benign first call sets a permissive context or passes a coarse check, and the second executes because per-entry authorization was never enforced.

## Error and ordering leaks

Batches also leak behavior through ordering and errors. Observing which entries succeed, which fail, and how errors are reported reveals whether authorization, validation, and side effects are evaluated per entry or once for the array, which guides where to place the restricted call.

## Confirming the flaw

Send a batch whose entries would be throttled or blocked individually and compare the outcome to sending them one at a time. If a per-request limit registers once for the whole batch, or a restricted method executes when paired with an allowed one, the controls are applied at the wrong granularity. The fix, authorizing and accounting for each entry independently and defining an atomic failure policy, is the property this test is checking for.

## References

- [JSON-RPC 2.0 Specification: Batch](https://www.jsonrpc.org/specification#batch)
- [OWASP API Security Top 10: Unrestricted Resource Consumption](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)
