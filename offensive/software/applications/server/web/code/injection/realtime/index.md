---
title: "Realtime injection"
description: "Long-lived channels where message bodies and metadata drive routing, authorization, and sinks, so individual frames carry injection past the HTTP request boundary."
keywords:
  - realtime injection
  - WebSocket
  - server-sent events
  - long polling
  - message injection
---

# Realtime

Realtime transports keep a channel open long after the initial HTTP request, and over that channel the application exchanges many small messages instead of discrete request/response pairs. WebSocket frames, Server-Sent Events, and long-polling cycles all share one property that matters for an attacker: the body and metadata of each message drive routing, authorization decisions, and downstream sinks, yet the machinery that inspects a normal HTTP request often runs only once, at setup.

That gap is the target. A frame that reaches a query builder, a broadcast buffer, a command runner, or a stream serializer is attacker-controlled input that frequently skips the WAF rules, auth middleware, and validation applied to REST routes. Per-message authorization is routinely weaker than per-request authorization, origins are often unchecked at the handshake, and newline-delimited stream formats invite record forging.

This subtree is organized by transport: WebSocket message handling, Server-Sent Events, and long polling.
