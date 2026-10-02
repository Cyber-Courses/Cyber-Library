---
title: "SSRF hostname and DNS: rebinding, IDN, and resolver behavior in server-side URL fetches"
description: Hostname and DNS-layer facets of outbound URL fetches—rebinding, IDN, and resolver behavior in application SSRF testing.
keywords:
  - SSRF
  - DNS rebinding
  - domain name
---

# Domain name (SSRF)

This node holds hostname-centric SSRF material: DNS rebinding where a name resolves first to a public IP and later to `127.0.0.1`, internationalized domain homoglyphs, and differences between what the application resolves and what you see from your workstation. Detailed leaves are added as the topic map grows.

## See also

- [Authority (parent)](../index.md)
