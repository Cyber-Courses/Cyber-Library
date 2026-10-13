---
title: "Traffic interception: capturing and hijacking cleartext Telnet"
order: 4
description: "Telnet is unencrypted, so a positioned attacker reads everything: the login and password, every command typed, and all output, and can hijack the live TCP session to inject commands as the authenticated user. Interception is often the easiest Telnet attack where a network position exists, needing no credential guessing."
keywords:
  - telnet interception
  - cleartext
  - password sniffing
  - session hijacking
  - mitm
---

# Traffic interception

Because Telnet encrypts nothing, an attacker with a network position gets everything for free: the username and password as they are typed at login, every command the user runs, and all the server's output. Beyond passive capture, the unauthenticated, unencrypted TCP session can be hijacked, injecting commands that execute as the already-authenticated user. Where a capture or on-path position exists, interception is usually the easiest and quietest Telnet attack, bypassing authentication entirely.

```bash
# capture Telnet from an on-path position
tcpdump -i eth0 -A port 23 -w telnet.pcap
# follow the TCP stream to read credentials, commands, and output
```

## Subtopics

- **[Password sniffing](password-sniffing.md)**: capturing credentials from the cleartext login.
- **[Command interception](command-interception.md)**: reading commands and output in transit.
- **[Session hijacking](session-hijacking.md)**: taking over the live Telnet session.

## References

- [RFC 854 (Telnet, no encryption)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
