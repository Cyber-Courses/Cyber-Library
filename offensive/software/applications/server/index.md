---
title: "Server"
description: Offensive testing of server-side application software—APIs, web applications, and supporting services—where request handling and trust boundaries are implemented in code.
keywords:
  - server application security
  - backend security
  - web server application
  - API security
---

# Server

**Server** applications accept requests, enforce policy in code, and persist or retrieve data on behalf of clients. Testing here emphasizes **application-layer** behavior: routes, handlers, identity, authorization, and integration with databases and other services.

## Web applications and APIs

- **[Web](web/index.md)** — HTTP-based services, including document-style sites and machine-facing APIs, with a subtree for **application code** flaws.

## Notes

Pure infrastructure misconfiguration (for example, only a load balancer TLS profile) may be documented elsewhere; this branch still hosts issues when the **application** must validate or use those components correctly (for example, trusting proxy headers in app code). See **[Web → Code](web/code/index.md)** for the split between app logic and platform.

## See also

- [Applications (parent)](index.md)