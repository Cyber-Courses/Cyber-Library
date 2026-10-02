---
title: "SSRF URL authority: host, port, IP literals, and DNS tricks in server-side fetches"
description: URL authority components in server-side fetches, hostnames, IP literals, ports, and DNS-related tricks that change where the application connects.
keywords:
  - SSRF
  - URL authority
  - DNS rebinding
---

# Authority (SSRF)

The **authority** of a URL is where the client connects: host (name or IP literal), optional port, and optional userinfo. SSRF testing varies encoding of IPs, use of `127.0.0.1`, link-local and metadata addresses, and parser differences for internationalized hostnames.

## Pages

- [IP address](ip-address.md), Literal forms, decimal/hex encodings, IPv6-mapped IPv4.
- [Port](port.md), Non-default ports and parser acceptance.
- [Domain name](domain-name/index.md), Hostname tricks and DNS timing (stub for expanded tree).
