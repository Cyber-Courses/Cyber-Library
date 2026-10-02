---
title: "SSRF fetch clients: programmatic HTTP, headless browsers, and document pipelines compared"
description: How the application performs outbound requests, programmatic HTTP clients, headless browsers, and document converters, and why the same URL behaves differently per stack.
keywords:
  - SSRF
  - HttpClient
  - headless browser
---

# Fetch client (SSRF)

The **fetch** facet is the code path that actually opens the socket: `HttpURLConnection`, `fetch()`, `requests.get()`, Playwright navigation, PDF renderers, and image loaders. Policy on “allowed hosts” is meaningless if a **second** code path uses a different client with different redirect limits, TLS validation, or scheme support.

## Pages

| Page | Focus |
|------|--------|
| [Programmatic HTTP client](programmatic-client.md) | Library stacks, redirects, connection pools |
| [Headless browser](headless-browser.md) | Playwright / Puppeteer navigation vs thin clients |
| [Document pipeline](document-pipeline.md) | PDF, previews, image proxies |
