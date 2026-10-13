---
title: "Realtime injection"
order: 11
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

## Subtopics

- **[WebSocket](websocket/index.md)**: Abusing WebSocket message handling after the HTTP upgrade, where individual frames drive authorization, routing, and server-side sinks.

## Pages

- **[Long polling](long-polling.md)**: Held-open and poll endpoints where JSON notifications, poll query params, and session tokens in the poll URL reach server-side queries and authorization.
- **[Server-Sent Events](server-sent-events.md)**: Newline injection into a text/event-stream where server-built data, event, and id lines come from untrusted state, forging events and poisoning shared streams.

## Tools

- **Burp Suite**: WebSockets history and repeater for intercepting and replaying frames.
- **websocat**: command-line WebSocket client for crafting raw messages.
- **curl -N**: stream a Server-Sent Events endpoint to inspect unparsed bytes.

## References

- PortSwigger Web Security Academy: WebSocket security vulnerabilities
- OWASP Web Security Testing Guide: Testing WebSockets
