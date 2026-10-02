---
title: "SSRF port selection in URL authority: non-default ports, redirect hops, and filter bypass"
description: Non-default and parser-edge ports in attacker-supplied URLs for server-side fetches.
keywords:
  - SSRF
  - port
  - URL authority
---

# Ports (SSRF)

## Context

The URL authority includes an optional port. Applications that block `http://127.0.0.1` sometimes forget `http://127.0.0.1:443` or `http://127.0.0.1:80` when the block list is string-based. Internal services often listen on high ports (Redis, Elasticsearch, admin UIs).

## Theory

Some stacks default missing ports; others preserve explicit `:0` or reject it. Redirect responses may change host and port together; the fetcher may re-apply policy on each hop or only the first URL—behavior is library-specific.

## Practice

### Sweep common internal ports on a fixed host

- In scope, supply `http://internal.service:PORT/path` through the SSRF sink and observe timing, error bodies, or blind side channels for open vs closed ports (respect rate and impact rules in the program).

### Test explicit :443 on loopback

- If filters block `127.0.0.1` without port, try `127.0.0.1:443` and `127.0.0.1:22` to see if the filter is naive substring-based.

## Tools

- **Burp Suite**
- **ffuf** (against your own lab)