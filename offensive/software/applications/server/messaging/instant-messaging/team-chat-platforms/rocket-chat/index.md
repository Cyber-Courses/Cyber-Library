---
title: "Rocket.Chat: attacking the /api/v1 REST server and its integration scripts"
description: "Attacking a self-hosted Rocket.Chat server: a Node/Meteor process on 3000 exposing the /api/v1 REST API and DDP over MongoDB. Fingerprint the build from /api/info and the Meteor /sockjs/ transport, then move through user and channel enumeration, the NoSQL operator-injection login bypass, and server exploitation by chaining that bypass to an admin token and the integration-script sandbox escape for code execution."
keywords:
  - rocket.chat
  - rocket.chat api
  - nosql injection login
  - integration script rce
  - meteor ddp
---

# Rocket.Chat

Rocket.Chat is a Node.js application built on the Meteor framework, serving a web client and a REST API rooted at `/api/v1` on port `3000` by default (commonly proxied to 443), with Meteor's DDP protocol over a `/sockjs/` websocket and MongoDB underneath. The MongoDB backing and Meteor's permissive handling of request bodies drive the signature attack: the login endpoint accepts a structured object where a string is expected, so a Mongo query operator injected into `user` or `password` turns authentication into a query that matches without the real password. Hold any admin token and the admin-only **integrations** feature runs server-side JavaScript in a sandbox that has repeatedly been escapable to `require`/`process`, which is the route to code execution as the Node user.

## Fingerprint first

Confirm the product and pin the build, because the login-bypass behavior and the integration-script sandbox differ by version.

```bash
curl -sk http://<target>:3000/api/info
#   {"version":"6.5.0","success":true}       confirms Rocket.Chat + exact version
curl -skI http://<target>:3000/sockjs/info   # Meteor/SockJS transport (DDP) present
curl -skI http://<target>:3000/ | grep -i x-instance-id   # X-Instance-Id header is Rocket.Chat
```

`/api/info` gives the version with no authentication on most builds; where it is locked down, the Meteor bootstrap data in the page source and the `__meteor_runtime_config__` blob still leak the release. Note whether `/api/v1/settings.public` is reachable unauthenticated (below) because it exposes whether registration is open and which login methods are enabled. Record the version before attacking the login layer.

## Pages

- **[Enumeration](enumeration.md)**: harvesting users and channels through `/api/v1/users.list`, `/api/v1/channels.list`, and `/api/v1/spotlight`, reading `settings.public`, and using the Meteor DDP methods; interpreting the JSON returned with `X-Auth-Token`/`X-User-Id` headers.
- **[Authentication](authentication.md)**: the NoSQL operator-injection login bypass where `POST /api/v1/login` and the Meteor `login` method accept an object for `user`/`password`, plus the password-reset token weakness, returning an `authToken` and `userId`.
- **[Server exploitation](server-exploitation.md)**: chaining the auth bypass to an admin token, then the incoming/outgoing integration "script" and webhook sandbox escape to `require`/`process` for code execution, the file-upload sink, and the message-parser SSRF; ending in a shell as the node user.

## References

- [Rocket.Chat REST API reference (`/api/v1`)](https://developer.rocket.chat/apidocs)
- [Rocket.Chat integrations and script documentation](https://docs.rocket.chat/docs/integrations)
- [RocketChat/Rocket.Chat (server source)](https://github.com/RocketChat/Rocket.Chat)
- [Meteor DDP protocol specification](https://github.com/meteor/meteor/blob/devel/packages/ddp/DDP.md)
