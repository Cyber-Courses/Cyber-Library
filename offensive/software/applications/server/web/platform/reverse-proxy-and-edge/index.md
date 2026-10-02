---
title: "Reverse proxy and edge: edge/origin mismatches, backend exposure, and header trust"
description: "Platform attacks that belong to the proxy role rather than one product: path normalization mismatch between edge and origin, reaching the origin directly, and header trust injected by the edge."
keywords:
  - reverse proxy
  - path confusion
  - origin exposure
  - x-forwarded
  - waf bypass
---

# Reverse proxy and edge

A reverse proxy or CDN/WAF in front of the origin adds a second request parser and a routing table, so security now depends on the proxy and origin agreeing about *what the request is* and *where it goes*. These issues belong to the proxy **role** rather than any single server product, which is why they sit here instead of under Apache/nginx/IIS (a product-specific proxy bug, like nginx variable `proxy_pass`, lives under its product).

## What to check

- **Normalization mismatch**: the edge and origin canonicalize paths differently, so a crafted path passes an edge allow/deny rule but resolves to a protected resource at the origin.
- **Origin exposure**: the backend is reachable directly, bypassing the proxy's filtering, rate limits, and access control entirely.
- **Edge header trust**: the proxy injects or forwards `X-Forwarded-*`/`Host` that the origin then trusts as authoritative.

## Related, covered elsewhere

- **HTTP request smuggling** (front/back desync on request boundaries) is in [Code > Injection > HTTP](../../code/injection/http/request-smuggling.md).
- The application-side consequences of trusting forwarded headers are in [Code > Access Control > proxy header trust](../../code/access-control/trust-boundary/proxy-header-trust-abuse.md) and [Identification](../../code/identification/rate-limits-and-automation.md); here the focus is the edge wiring that creates that trust.

## Pages

- **[Normalization mismatch](normalization-mismatch.md)**: edge and origin disagree on the canonical path, bypassing path-based access control.
- **[Origin exposure](origin-exposure.md)**: reaching the backend directly, past the proxy, WAF, or CDN.
- **[Edge header trust](edge-header-trust.md)**: how proxy wiring makes forged `X-Forwarded-*`/`Host` headers authoritative.

## References

- PortSwigger Web Security Academy: Access control, SSRF, Web cache
- Orange Tsai: reverse-proxy path confusion research
