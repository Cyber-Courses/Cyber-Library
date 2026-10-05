---
title: "IRC: attacking Internet Relay Chat servers"
description: "IRC on TCP 6667 plaintext and 6697 over TLS, served by ircd implementations such as UnrealIRCd, InspIRCd, and Charybdis. The surface is unauthenticated enumeration through registered sessions, NickServ/ChanServ services and the OPER operator privilege, and server-side flaws including the backdoored UnrealIRCd 3.2.8.1 distribution."
keywords:
  - IRC
  - ircd
  - port 6667
  - IRC operator
  - UnrealIRCd
---

# IRC

Internet Relay Chat is a line-based text protocol on TCP 6667 (plaintext) and 6697 (TLS). A server is an ircd (UnrealIRCd, InspIRCd, Charybdis, ngIRCd, ircd-hybrid) that hosts channels, links to other servers to form a network, and runs companion services (NickServ, ChanServ) for account and channel ownership. Two privilege tiers matter offensively: a normal registered user who can enumerate the whole network, and an IRC operator (oper) who holds server commands up to loading modules and executing on the host. The channels themselves frequently carry credentials, bot command interfaces, and internal chatter.

A session is trivial to open, and the first lines already fingerprint the daemon:

```bash
nmap -p6667,6697 -sV --script irc-info,irc-unrealircd-backdoor <target>

nc <target> 6667
NICK pentest
USER pentest 0 * :pentest
# the server replies with numerics: 001 (welcome), 002/003/004 (server + ircd version),
# 005 (ISUPPORT feature list). 004 names the daemon, e.g. "UnrealIRCd-6" or "InspIRCd-3".
```

A `464 :Password required` in response means the server has a connection password (`PASS`), and an immediate `ERROR :Closing link` with a K-line message means your source address is banned.

## Triage

- Does the handshake complete without a `PASS`, and what does numeric `004`/`005` say the ircd and version are? That decides whether a known server flaw applies.
- Can you `LIST` channels and `WHO *`/`WHOIS` users without being an operator? That drives [enumeration](enumeration.md).
- Are NickServ and ChanServ present, and can you reach operator status via `OPER`? That drives [services and operator abuse](services-and-operator-abuse.md).
- Is the daemon a version with a known backdoor or module flaw? That drives [server exploitation](server-exploitation.md).

## Subtopics

- **[Enumeration](enumeration.md)**: register a session, then read the network with `VERSION`, `LUSERS`, `LIST`, `WHO`, `WHOIS`, `NAMES`, `STATS`, and `MAP`, plus service queries.
- **[Services and operator abuse](services-and-operator-abuse.md)**: NickServ/ChanServ takeover, SASL, and escalation to IRC operator via `OPER` and weak O:lines.
- **[Server exploitation](server-exploitation.md)**: the backdoored UnrealIRCd 3.2.8.1 distribution, InspIRCd and Charybdis module and TLS issues, and DCC abuse.

## References

- [RFC 1459 (Internet Relay Chat Protocol)](https://datatracker.ietf.org/doc/html/rfc1459)
- [RFC 2812 (IRC Client Protocol)](https://datatracker.ietf.org/doc/html/rfc2812)
- [HackTricks: pentesting IRC (6667)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-irc.html)
