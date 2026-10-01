---
title: "WebSocket injection"
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
