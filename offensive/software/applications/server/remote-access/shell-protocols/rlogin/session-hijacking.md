---
title: "Session hijacking: taking over a live rlogin session"
order: 2
description: "An rlogin session is an unencrypted TCP stream authenticated only at setup, so an on-path attacker injects commands into it or takes it over, executing as the logged-in user. This is the classic r-command/Telnet TCP session-hijacking attack, bypassing authentication by riding an established session."
keywords:
  - session hijacking
  - rlogin
  - tcp injection
  - hunt
  - juggernaut
---

# Session hijacking

Once an rlogin session is established, it is an unencrypted TCP stream with no cryptographic per-packet authentication, so an on-path attacker may attempt to hijack it. By reading the stream to learn the TCP sequence, acknowledgement, and window state, the attacker can inject segments carrying commands that the server executes as the logged-in user. Injected bytes advance one endpoint's sequence space, so continued control requires suppressing or rewriting competing traffic, translating sequence numbers, or accepting that the legitimate connection will desynchronize. This bypasses application login because the attacker commandeers a session another user already authenticated, running with that user's privileges. `rlogin` was a canonical target of classic session-hijacking tools.

```bash
# on-path position to the rlogin session (ARP/route); the cleartext stream reveals
# the TCP seq/ack state. Inject a command segment the server runs as the user:
#   e.g. append trust for durable re-entry
#   echo "+ +" >> ~/.rhosts
# classic automated tools (Hunt, Juggernaut) implemented rlogin/telnet TCP hijacking;
# modern exploitation crafts the segments directly from the observed seq/ack.
```

## Exploitation notes

- Injecting a single command (for example, planting `.rhosts`) may be enough to establish persistence, but it is not inherently quiet: unless subsequent traffic is adjusted or suppressed, the forged bytes desynchronize the legitimate endpoint and can expose the attack.
- The injected command runs as the logged-in user, often administrative on legacy Unix, so one command can be a full compromise.
- It needs only an on-path position and the cleartext stream, no credential or trust; it is the active counterpart to passively [capturing the session](cleartext-passwords.md).
- The identical technique applies to rsh and Telnet; the shared cleartext, setup-only-auth design is the root cause.

## References

- [HackTricks: rlogin hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
