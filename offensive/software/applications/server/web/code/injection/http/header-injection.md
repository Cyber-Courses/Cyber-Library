---
title: "HTTP header injection: Host and forwarding headers"
description: "Abusing attacker-controlled HTTP headers the application trusts: Host header for password-reset poisoning and cache poisoning, and X-Forwarded / override headers for access-control bypass and SSRF."
keywords:
  - host header injection
  - password reset poisoning
  - X-Forwarded-For
  - X-Forwarded-Host
  - X-Original-URL
---

# Header injection

Applications often trust request headers that the client fully controls. When a header value feeds routing, access control, or generated links, setting it to an attacker value subverts that logic. No CRLF is needed here; the abuse is in the value itself (CRLF response splitting is covered separately).

The `Host` header is the classic target because applications reuse it to build absolute URLs. A password-reset flow that builds the reset link from `Host` sends the victim a link pointing at the attacker's domain, leaking the token when clicked:

```
POST /reset HTTP/1.1
Host: attacker.tld
...
email=victim@example.com
```

The same reflected `Host` enables web cache poisoning (a cached page with an attacker-controlled absolute resource URL) and, where virtual-host routing or access rules key on `Host`, routing and ACL bypass. When the front end pins `Host`, the overrides `X-Forwarded-Host` and `X-Host` often reach the application and are honored instead.

Forwarding and override headers carry trust the attacker can forge. `X-Forwarded-For` is believed for IP-based access control and rate limiting, so spoofing it bypasses IP allowlists or resets counters. `X-Forwarded-Host`/`X-Forwarded-Proto` influence link and redirect generation. `X-Original-URL` and `X-Rewrite-URL` make some stacks route to a different path than the one the front-end access control checked, bypassing path-based restrictions (requesting `/` with `X-Original-URL: /admin`).

The test is to send each header with an attacker value and watch for it in a link, a routing decision, or an authorization outcome. Reach depends on the framework and proxy chain, so confirm which headers the back end actually honors.

## Tools

- **Burp Suite / Burp Repeater**: inject `Host` and forwarding/override headers and watch links, routing, and auth outcomes.
- **curl**: send arbitrary `Host`, `X-Forwarded-*`, and `X-Original-URL` headers from the command line.
- Manual testing with Burp Repeater and crafted payloads.

## References

- PortSwigger Web Security Academy: HTTP Host header attacks
- OWASP Testing Guide: Testing for Host Header Injection
