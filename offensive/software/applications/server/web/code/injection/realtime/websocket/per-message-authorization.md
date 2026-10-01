---
title: "Per-message authorization"
description: "WebSocket connections authorized only at the handshake, so individual frames are trusted and can act as other users or invoke privileged actions."
keywords:
  - per-message authorization
  - WebSocket authorization
  - IDOR WebSocket
  - broken access control
  - frame tampering
---

# Per-message authorization

Many WebSocket servers authenticate and authorize once, during the upgrade handshake, then treat every frame on the resulting connection as belonging to that authorized principal. The handler reads an action name and a set of identifiers out of each frame and acts on them, trusting that whoever owns the socket is allowed to touch whatever the frame names. That assumption is the opening: the identity was checked, but the specific object, room, or privileged operation named in each frame never is.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

## Switching the object identifier

A chat or collaboration socket typically carries a document, conversation, or resource id in each frame. If the handler loads that id without re-checking that the connected user owns it, incrementing or swapping the value reaches other users' data over the same authenticated connection.

```json
{"action": "message.list", "conversationId": 4021}
```

Replay with a neighboring id and read a conversation that belongs to someone else:

```json
{"action": "message.list", "conversationId": 4022}
```

The same pattern applies to writes. A frame that posts as the connected user often carries a `senderId` or `authorId` the server trusts verbatim:

```json
{"action": "message.send", "conversationId": 4022, "senderId": 7, "body": "approved, release the funds"}
```

Setting `senderId` to another account forges a message attributed to that user, because the handshake identity is never reconciled against the body field.

## Switching the room or channel

Pub/sub servers map a socket to channels through subscribe frames. If subscription is not authorized per channel, a low-privilege connection can join an administrative or private stream and receive everything broadcast to it:

```json
{"action": "subscribe", "room": "admin:alerts"}
```

```json
{"action": "subscribe", "room": "tenant:acme:billing"}
```

Once subscribed, the attacker is a passive recipient of every event the server publishes to that room, which often includes other tenants' records in a multitenant deployment.

## Invoking privileged actions

Servers frequently multiplex both ordinary and administrative operations onto a single connection, gating the admin ones behind a UI the non-admin client never renders. The socket still accepts the frames. Enumerating action names reaches operations the account was never meant to call:

```json
{"action": "user.setRole", "userId": 7, "role": "admin"}
```

```json
{"action": "config.update", "key": "signups.open", "value": true}
```

Because the per-message layer checks only that the connection is authenticated, not that this principal may perform this action on this target, each of these frames executes with the same authority as the legitimate admin console.

## Finding the action surface

Capture the client's own traffic to learn the frame schema, then mutate it. Browser developer tools expose every frame under the network panel's WS view, and the same frames can be replayed and fuzzed from a standalone client:

```javascript
// run in the authenticated user's browser console; the handshake sends cookies automatically
const ws = new WebSocket('wss://target.example/stream');
ws.onopen = () => {
  for (let id = 4000; id < 4100; id++) {
    ws.send(JSON.stringify({ action: 'message.list', conversationId: id }));
  }
};
ws.onmessage = (e) => console.log(e.data);
```

From a standalone Node `ws` client there is no ambient cookie, so pass a captured session string explicitly: `new WebSocket(url, { headers: { Cookie: 'session=...' } })`. Frames that return data for ids outside the connected user's own set confirm that authorization lives only at the handshake.

## References

- PortSwigger Web Security Academy, "Testing for WebSockets security vulnerabilities"
- OWASP Web Security Testing Guide, "Testing WebSockets"
