---
title: "SSRF and query parameter pollution: duplicate keys, first-wins vs last-wins, and outbound URL builders"
description: Duplicate or conflicting query keys when applications build outbound URLs from partially user-controlled fragments.
keywords:
  - SSRF
  - parameter pollution
---

# Parameter pollution (SSRF)

## Context

If the application concatenates base URL, path, and query from different sources, duplicate keys (`next=`, `url=`, `dest=`) may let the last or first value win in ways the developer did not intend—changing the effective host the HTTP client requests after parsing.

## Theory

First-wins vs last-wins depends on the URL builder and the HTTP stack. Compare behavior of Node `URL`, Python `urllib`, and Java `URI` in code review when the same pattern appears in multiple parameters.

## Practice

- Fuzz duplicate keys and ordering on callback parameters in a staging environment; diff final request URL from server logs if available.

## Tools

- **Burp Suite**