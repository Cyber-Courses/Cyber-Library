---
title: "Instant messaging: attacking chat protocols and team-chat platforms"
description: "Attacking real-time messaging infrastructure: the open chat protocols with multiple server implementations (IRC, XMPP, Matrix) and the self-hosted team-chat platforms reached as web applications and APIs (Mattermost, Rocket.Chat)."
keywords:
  - instant messaging
  - chat server
  - IRC
  - XMPP
  - Matrix
  - Mattermost
  - Rocket.Chat
---

# Chat

Real-time messaging servers split into two attack models. The open protocols (IRC, XMPP, Matrix) are attacked on the wire: you speak the protocol to enumerate the server, abuse its services and federation, and exploit the daemon. The self-hosted team-chat platforms (Mattermost, Rocket.Chat) are attacked as web applications: REST and websocket APIs where the weaknesses are authentication bypass, authorization gaps, injection, and integration or plugin code execution.

## Triage

```bash
nmap -sV -p6667,6697,5222,5223,5269,8008,8448,8065,3000 <target>
#  6667/6697 IRC   5222/5269 XMPP   8008/8448 Matrix   8065 Mattermost   3000 Rocket.Chat
curl -s http://<target>:8065/api/v4/system/ping        # Mattermost
curl -s http://<target>:3000/api/info                  # Rocket.Chat
curl -s https://<target>/_matrix/client/versions       # Matrix homeserver
```

An IRC/XMPP/Matrix port routes to [Protocols](protocols/index.md); a Mattermost or Rocket.Chat web app routes to [Team-chat platforms](team-chat-platforms/index.md).

## Subtopics

- **[Protocols](protocols/index.md)**: IRC, XMPP, and Matrix, attacked on the wire from enumeration through server exploitation.
- **[Team-chat platforms](team-chat-platforms/index.md)**: self-hosted Mattermost and Rocket.Chat, attacked through their web APIs.

## References

- [RFC 1459: Internet Relay Chat](https://www.rfc-editor.org/rfc/rfc1459)
- [RFC 6120: XMPP Core](https://www.rfc-editor.org/rfc/rfc6120)
- [Matrix specification](https://spec.matrix.org/latest/)
