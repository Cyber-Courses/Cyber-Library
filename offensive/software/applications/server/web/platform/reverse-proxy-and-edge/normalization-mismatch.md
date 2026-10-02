---
title: "Proxy normalization mismatch: path confusion that bypasses access control"
description: "Exploiting different path canonicalization between a reverse proxy and its origin: encoded slashes, dot-segments, and path parameters that pass an edge allow/deny rule while resolving to a protected path at the origin."
keywords:
  - path confusion
  - normalization mismatch
  - reverse proxy bypass
  - encoded slash
  - access control bypass
---

# Proxy normalization mismatch

A reverse proxy often enforces access rules on the request path ("deny `/admin`", "require auth under `/internal`") and then forwards to the origin. If the proxy and the origin **canonicalize the path differently**, a crafted path looks allowed to the proxy but resolves to the protected resource at the origin, bypassing the control.

## The mechanism

The proxy matches its rule against one interpretation of the path; the origin maps a different interpretation to a file or route. Common divergences:

- **Encoded slashes/dots**: the proxy does not decode `%2f`/`%2e` but the origin does, so `/public/..%2fadmin` passes a "deny /admin" rule and resolves to `/admin`.
- **Path parameters**: `/admin;foo=bar` or `/..;/admin` are distinct to the proxy but stripped by a Java/Tomcat origin (see [servlet path parameters](../tomcat-and-java/path-parameter-and-normalization.md)).
- **Dot-segment timing**: one tier collapses `/a/../admin` to `/admin` and the other does not.
- **Trailing content**: `/admin/..%2f..%2fadmin`, double slashes, or a trailing `/.` that only one side normalizes.

```
GET /public/..%2fadmin/ HTTP/1.1          # edge sees /public/..%2fadmin, origin sees /admin
GET /admin..;/ HTTP/1.1
GET /%2561dmin/ HTTP/1.1                   # stacked with double-encoding
```

## Finding it

- Identify a path the edge protects (401/403/redirect at `/admin`, `/internal`, `/actuator`, `/manager`).
- Replay it wrapped in forms the edge is unlikely to normalize but the origin will: `..%2f`, `;/`, `%2e%2e`, double slashes, mixed case, and combinations.
- A protected-resource body returning `200` through a crafted path confirms the mismatch.

## Exploitation

- Reach admin consoles, framework actuators/management endpoints, and internal APIs the edge believed it had fenced off (including the [Tomcat Manager](../tomcat-and-java/manager-and-host-manager.md)).
- Combine with [origin exposure](origin-exposure.md): if the control exists only at the edge, a desync to the protected route has the same effect as talking to the origin directly.
- Stack with [double decoding](../iis/double-decode-and-unicode-traversal.md) when the edge decodes once.

## Tools

- Burp (Repeater/Intruder) with path-confusion payloads; **curl --path-as-is**.

## References

- PortSwigger Web Security Academy: Access control bypass
- Orange Tsai: reverse-proxy path confusion research
