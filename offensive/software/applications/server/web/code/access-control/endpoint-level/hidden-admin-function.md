---
title: "Hidden admin functions: undisclosed routes, feature flags, and operator-only actions"
description: Undocumented HTTP routes and RPC methods that still exist in production, often with weaker review or missing authorization.
keywords:
  - undocumented API
  - internal route
  - legacy endpoint
  - debug API
---

# Hidden admin functions

## Context

These are not necessarily malicious backdoors. They are often support hooks, one-off migration endpoints, or “temporary” admin tools that never left the tree. They skip public OpenAPI and sometimes skip the same review bar as first-class routes.

## Theory

Recon combines public spec gaps with mobile binary strings, error traces, and GraphQL field sets that do not appear in the marketing client. The failure mode is still missing or weak function-level authorization on a powerful handler.

## Practice

### Cross mobile and web surface

- Compare the list of paths or operationIds from a mobile binary’s hardcoded list to the web app’s network calls. Mismatches flag server routes only some clients use.

## Tools

- **jadx** / **apktool** (for Android lab artifacts in scope)
- **Burp Suite**
- **strings**
