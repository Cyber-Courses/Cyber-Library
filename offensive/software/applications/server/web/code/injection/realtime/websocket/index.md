---
title: "WebSocket injection"
order: 1
description: "Abusing WebSocket message handling after the HTTP upgrade, where individual frames drive authorization, routing, and server-side sinks."
keywords:
  - WebSocket injection
  - WebSocket frame
  - wss
  - message handling
  - realtime abuse
---

# WebSocket

A WebSocket starts life as an HTTP request with an `Upgrade: websocket` header, but once the server answers `101 Switching Protocols` the connection leaves the request/response model entirely. From then on both sides push frames at will, and the server dispatches each frame through a message handler that decides what it means and what it touches.

That handler is where the interesting attack surface lives. The handshake may have been authenticated and origin-checked, yet each frame after it is often trusted implicitly: the server reads an `action`, an `id`, or a `room` out of the frame and acts, without re-running the checks that guarded the upgrade. Frame bodies also flow into the same backends as any other input, so a frame can carry a SQL fragment, a script payload, or a template expression straight to a sink while bypassing the HTTP-layer filtering that would have caught it on a REST route.

This section covers per-message authorization gaps, cross-site WebSocket hijacking, and message injection into downstream sinks.

## Pages

- **[Cross-site WebSocket hijacking](cross-site-websocket-hijacking.md)**: A WebSocket handshake that relies on ambient cookies and skips Origin validation lets an attacker page open an authenticated socket as the victim.
- **[Message injection to sinks](message-injection-to-sinks.md)**: WebSocket frame content flowing unsanitized into SQL, NoSQL, stored XSS broadcast, and command or template sinks, carried past HTTP-layer filtering.
- **[Per-message authorization](per-message-authorization.md)**: WebSocket connections authorized only at the handshake, so individual frames are trusted and can act as other users or invoke privileged actions.

## Tools

- **Burp Suite**: WebSockets history and repeater for intercepting, editing, and replaying frames.
- **wsrepl**: interactive WebSocket REPL built for penetration testing.
- **websocat**: command-line WebSocket client for scripting and fuzzing raw frames.

## References

- PortSwigger Web Security Academy: WebSocket security vulnerabilities
- OWASP Web Security Testing Guide: Testing WebSockets
