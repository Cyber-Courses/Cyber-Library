---
title: "Protocols: attacking open real-time chat protocols and their servers"
description: "Open real-time chat protocols with multiple independent server implementations: IRC on 6667/6697, XMPP on 5222/5223/5269, and Matrix on 8008/8448/443. Each exposes a self-service surface (enumeration, account provisioning, authentication abuse) and implementation-specific server flaws. Covers how to fingerprint which protocol and daemon is running and where to go from there."
keywords:
  - instant messaging protocols
  - IRC
  - XMPP
  - Matrix
  - chat server
---

# Protocols

These are the open, standardized real-time chat protocols, each with several independent server implementations rather than a single vendor product. That shape drives the offensive approach: the protocol is public, so you speak it directly over a socket, and the value is in self-service surface (open registration, roster and directory enumeration, mechanism downgrade) plus bugs specific to whichever daemon is deployed. All three protocols federate or link servers, so a single compromised node often reaches a wider network of users and rooms.

## Triage

One sweep tells you which protocol, and the banner or stream handshake tells you which daemon:

```bash
nmap -sV -p6667,6697,5222,5269,8448 <target>
#  6667 IRC plaintext   6697 IRC over TLS
#  5222 XMPP client (STARTTLS)   5223 XMPP legacy TLS   5269 XMPP server-to-server
#  8008 Matrix client-server     8448 Matrix federation     443 Matrix behind a reverse proxy
```

Fingerprint by speaking the protocol:

- IRC answers a raw `NICK`/`USER` handshake with numeric replies and a `VERSION` reply naming the ircd (UnrealIRCd, InspIRCd, Charybdis, ngIRCd). See [IRC](irc/index.md).
- XMPP answers an opening `<stream:stream>` with a `<stream:features>` block whose `from` and namespaces name the server (ejabberd, Prosody, Openfire). See [XMPP](xmpp/index.md).
- Matrix answers `GET /_matrix/client/versions` and `/_matrix/federation/v1/version` with JSON naming Synapse, Dendrite, or Conduit. See [Matrix](matrix/index.md).

For each, the recurring questions are the same: can you enumerate users, rooms, and server features without credentials; can you self-provision or authenticate as someone; and does the specific daemon have a known server-side flaw.

## Subtopics

- **[IRC](irc/index.md)**: text protocol on 6667/6697, channels, NickServ/ChanServ services, and IRC operators, with backdoored and vulnerable ircd builds.
- **[XMPP](xmpp/index.md)**: XML streams on 5222/5223/5269, service discovery, SASL and in-band registration, and server flaws in ejabberd, Prosody, and Openfire.
- **[Matrix](matrix/index.md)**: HTTP/JSON API on 8008/8448/443, open registration and shared-secret admin, media SSRF, and federation abuse against Synapse, Dendrite, and Conduit.

## References

- [RFC 1459 (Internet Relay Chat Protocol)](https://datatracker.ietf.org/doc/html/rfc1459)
- [RFC 6120 (XMPP Core)](https://datatracker.ietf.org/doc/html/rfc6120)
- [Matrix specification](https://spec.matrix.org/latest/)
- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
