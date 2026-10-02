---
title: "WebSocket, SSE, and long-poll: realtime channel authorization and message injection"
description: Long-lived channels where message routing and authorization apply per frame or event, not only at HTTP connect time.
keywords:
  - WebSocket
  - SSE
  - long polling
---

# Realtime channels

**Realtime** transports deliver user-influenced payloads to the same SQL, command, and file sinks as REST handlers, with a per-message dispatch path. Connect-time auth only is a common gap; each message may need the same object and field checks as a synchronous API.

## Pages

| Page | Focus |
|------|--------|
| [WebSocket per-message authorization](websocket-per-message-authorization.md) | BOLA on frames |
| [SSE and long poll](sse-and-long-poll.md) | Caching and session scope |
| [Message injection to sinks](message-injection-to-downstream-sinks.md) | SQL/command/file via realtime JSON |
