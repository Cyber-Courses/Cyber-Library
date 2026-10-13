---
title: "IIS misconfiguration: NTFS filename tricks, decoding, handlers, and web.config"
order: 4
description: "IIS-specific platform misconfigurations: NTFS alternate data streams and short-name enumeration, double-decode and Unicode traversal, handler mappings, and web.config behavior and disclosure."
keywords:
  - iis misconfiguration
  - ::$DATA
  - short name
  - double decode
  - web.config
---

# IIS

IIS on Windows/NTFS inherits filesystem quirks and a handler-and-config model that produce a distinctive misconfiguration set. Fingerprint it from `Server: Microsoft-IIS`, the `ASP.NET_SessionId` cookie, `X-AspNet-Version`/`X-Powered-By: ASP.NET`, and default error pages, then work this checklist.

## What to check

- **NTFS filename tricks**: `::$DATA` alternate data streams, trailing dots/spaces, and 8.3 short-name enumeration that disclose source or reveal hidden files.
- **Decoding**: double-decode and overlong Unicode traversal on older IIS, where the path is decoded more than once.
- **Handlers and web.config**: handler mappings that execute uploads or disclose source, and `web.config` behavior (per-directory config, source exposure, and upload-as-config abuse).

## Pages

- **[NTFS filename tricks](ntfs-filename-tricks.md)**: `::$DATA`, trailing dot/space, and short-name enumeration.
- **[Double-decode and Unicode traversal](double-decode-and-unicode-traversal.md)**: multi-stage and overlong decoding that smuggles traversal.
- **[Handlers and web.config](handlers-and-web-config.md)**: handler mappings, source disclosure, and `web.config` abuse.

## References

- Microsoft IIS documentation; NTFS alternate data streams and 8.3 naming
- Soroush Dalili: IIS short-name and ::$DATA research
