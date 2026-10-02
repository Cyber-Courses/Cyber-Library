---
title: "Server-side request forgery (SSRF): steering outbound fetches to internal hosts, metadata URLs, and non-HTTP schemes"
description: Outbound HTTP or other fetches in application code steered by attacker-influenced URLs—internal hosts, metadata endpoints, alternate schemes.
keywords:
  - SSRF
  - server side request forgery
  - URL parser
---

# SSRF

**SSRF** is the class where application code issues an outbound fetch (`HttpClient`, `requests`, `fetch`, headless browser navigation, and similar) to a URL or address the user partially controls, reaching hosts or schemes the user could not call directly from their own machine.

The topic tree splits the URL into **Authority** (host, port, DNS tricks), **Path**, **Query**, **Scheme**, and **Fetch** (programmatic HTTP client vs headless browser).

| Area | Start here |
|------|------------|
| Authority | [Authority](authority/index.md) — host, port, DNS rebinding |
| Path | [Path](path/index.md) — encoding, normalization, path parameters |
| Query | [Query](query/index.md) — redirects, parameter pollution |
| Scheme | [Scheme](scheme/index.md) — `gopher`, `file`, `ldap`, and related handlers |
| Fetch | [Fetch client](fetch/index.md) — programmatic HTTP vs headless browser |

## See also

- [Injection (parent)](../index.md)
- [HTTP (inbound)](http/index.md) — Not the same as outbound fetch abuse.
