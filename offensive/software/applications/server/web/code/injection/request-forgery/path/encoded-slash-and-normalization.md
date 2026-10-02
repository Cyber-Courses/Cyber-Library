---
title: "Encoded slashes and path normalization in SSRF URL building"
description: %2F versus literal slash, double encoding, and Unicode normalization changing the path the HTTP client requests versus what a filter inspected.
keywords:
  - SSRF
  - URL encoding
---

# Encoded slashes

## Context

Some stacks normalize `%2f` to `/` **before** host checks; others compare the **raw** string. **Double-encoding** and **path join** with user fragments can produce a **different** final path on the wire than the allowlist regex saw.
