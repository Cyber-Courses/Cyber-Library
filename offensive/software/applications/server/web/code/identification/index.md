---
title: "Account identification and discovery in web apps: usernames, emails, and enumerable identifiers"
description: Server-side signals that make account existence, identifiers, or rate limits easy to abuse for enumeration and credential attacks.
keywords:
  - account enumeration
  - credential stuffing
  - user enumeration
---

# Identification

This area covers how **application** behavior reveals whether accounts exist, eases **credential stuffing**, or exposes identifiers across systems: distinct error messages, timing differences, per-IP-only throttling, and public ids in APIs or exports. It sits next to [Authentication](../authentication/index.md) and [Access control](../access-control/index.md) but centers on **discovery** and **signal** quality, not only BOLA on object ids.

## Pages

| Page | Focus |
|------|--------|
| [Account enumeration](account-enumeration.md) | Errors, timing, registration and reset branches |
| [Rate limits and automation](rate-limit-and-automation.md) | Throttling gaps and stuffing surfaces |
| [Identifier exposure](identifier-exposure-in-urls-and-exports.md) | IDs in URLs, exports, and public fields |
