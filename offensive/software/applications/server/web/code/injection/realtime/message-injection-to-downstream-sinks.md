---
title: "Realtime message injection into SQL, command, and file sinks"
description: User payloads delivered over WebSocket or SSE that reach the same dangerous primitives as REST handlers.
keywords:
  - WebSocket
  - injection
---

# Message injection to sinks

## Context

JSON over WebSocket often feeds the same **service layer** as HTTP. **SQL**, **command**, and **path** injection classes apply when message handlers build strings unsafely.

## See also

- [Realtime channels (parent)](index.md)
- [Injection (parent)](../index.md)
