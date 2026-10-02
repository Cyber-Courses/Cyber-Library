---
title: "file:// URLs in SSRF: local file read via server-side fetchers, headless browsers, and document pipelines"
description: Local file read when the server-side URL client allows file scheme handlers and passes attacker-influenced paths.
keywords:
  - SSRF
  - file URI
  - local file read
---

# file:// (SSRF)

## Context

If `file:///etc/passwd` is accepted by the fetch API, the response body may return local filesystem bytes to the attacker through error messages, response forwarding, or PDF/image processors. Some stacks disable `file` by default; others enable it in headless browsers or document converters.

## Theory

Path normalization (`file:////`, Windows drive letters) varies. Chaining with path traversal in the injected string can reach sensitive paths when the handler is naive.

## Practice

- In a lab JVM or Node app, test `file:///etc/hosts` through the same URL parameter used for HTTP callbacks.

## Tools

- **Burp Suite**