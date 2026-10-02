---
title: "Programmatic HTTP clients in SSRF: redirects, connection reuse, and library-specific URL handling"
description: Server-side SSRF through HttpURLConnection, requests, fetch(), and language HTTP stacks, behavior that differs from curl and browsers.
keywords:
  - SSRF
  - HttpClient
  - redirects
---

# Programmatic HTTP client

Application code usually calls a **library** HTTP client with a URL string. Redirect limits, cookie jars, TLS verification, and **per-hop** URL filtering differ from **headless browsers** and from command-line **curl**. A policy that wraps only one call site misses alternate clients (webhooks, batch jobs, imports).

## Practice

- Inventory **every** outbound HTTP entry point (SDKs, job runners, microservice clients) and note redirect and scheme behavior for the same SSRF payload.
