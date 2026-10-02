---
title: "Headless browser SSRF: Playwright, Puppeteer, and full navigation versus API fetches"
description: Outbound requests from headless Chromium or WebKit, different origin rules, resource loading, and URL schemes than thin HTTP clients.
keywords:
  - SSRF
  - headless browser
  - Playwright
---

# Headless browser SSRF

When the “fetch” is **`page.goto(url)`** or loading `<img src>`, the stack may enable **cookies**, **JavaScript redirects**, **file** or **chrome-extension** schemes, and **subresource** loads. SSRF policy written for `requests.get` may not apply to the automation path.
