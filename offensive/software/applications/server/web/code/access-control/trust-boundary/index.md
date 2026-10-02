---
title: "Application trust boundaries: proxy headers, TLS client certs, and upstream identity"
description: When server-side code trusts client-supplied or upstream identity signals: forwarded headers, mTLS identity mapping, or proxy context, without sound binding to the request.
keywords:
  - trust boundary
  - forwarded header abuse
  - reverse proxy
  - client certificate
---

# Trust boundary

**Trust boundary** failures happen when **application** code makes authorization or routing decisions from data an attacker can influence because it is misinterpreted as coming from a trusted path: HTTP headers in front of an unsecured reverse hop, mTLS client identity that does not map uniquely to a user, or a shared secret between services that is guessable or leaked.

Inbound HTTP smuggling (parsing mismatch between hops) is a different family under **Injection → HTTP**; this page is “who does the app think this request is?” when the answer comes from **headers or certificates** the app must not trust naively from the Internet.

## Pages

- [Proxy header trust abuse](proxy-header-trust-abuse.md)
- [Client certificate misbinding](client-certificate-misbinding.md)

