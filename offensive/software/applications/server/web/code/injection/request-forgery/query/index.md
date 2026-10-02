---
title: "SSRF query string: redirects, duplicate parameters, and outbound URL construction bugs"
description: Query parameters in outbound URL fetches, redirect chains, parameter pollution, and filter bypass on the query facet of SSRF.
keywords:
  - SSRF
  - query string
  - open redirect
---

# Query string (SSRF)

The query facet covers what happens after `?` in the URL: attacker-controlled keys and values, open redirects that change the final destination when the HTTP client follows redirects, and parameter pollution when duplicate keys change server-side URL construction.

## Pages

- [Bypassing using a redirect](bypassing-using-a-redirect.md)
- [Parameter pollution](parameter-pollution.md)
