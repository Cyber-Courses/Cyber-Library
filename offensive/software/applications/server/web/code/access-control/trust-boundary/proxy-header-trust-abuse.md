---
title: "Proxy header trust abuse: X-Forwarded-For, X-Real-IP, and forged client addresses"
description: Backends that trust X-Forwarded-For, X-User-Id, or similar from a client that can reach the app without a real trusted forwarder.
keywords:
  - X-Forwarded-For
  - X-User-Id
  - trusted proxy
  - header spoofing
---

# Proxy header abuse

## Context

Reverse proxies set `Forwarded`, `X-Forwarded-For`, and sometimes app-specific identity headers. If the app container is exposed to the internet and still trusts those headers, a client can set `X-User-Id: 1` or a loopback `X-Forwarded-For` to influence **who the app thinks the caller is** or what IP the app logs for **rate** decisions.

## Theory

The trust model requires a first hop that strips or overwrites client-supplied forward headers. When that hop is missing, the application’s use of `req.ip` or `req.userId` from headers is attacker-controlled. Similar patterns apply to `X-Original-URL` on some stacks when used for routing decisions in mis-setups.

## Practice

### Send identity headers on a direct-to-app request in a lab

- In a test deployment that mirrors a bad layout, `curl` the app port with `X-User-Id` or `X-Role` and an otherwise unauthenticated or low-session request. Observe whether behavior changes relative to a request without those headers.

## Tools

- **curl**
- **Burp Suite**
