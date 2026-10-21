---
title: "Live session hijacking: injecting into an established r-command stream"
description: "An on-path attacker who observes an active, unencrypted r-command connection may inject correctly sequenced TCP data as the authenticated user. This is live TCP session hijacking, not replay of a saved exchange into a new connection."
keywords:
  - session hijacking
  - tcp hijacking
  - command injection
  - rsh
  - rlogin
---

# Live session hijacking

The r-commands authenticate during connection setup and then exchange application data over unencrypted TCP streams. An attacker who remains on path can observe the current TCP sequence, acknowledgement, and window state and attempt to inject data into a live connection. What that input means depends on the service: an `rlogin` session normally feeds an interactive shell, whereas an `rsh` or `rexec` primary stream feeds stdin to the command selected during setup. Injected text becomes a new shell command only when that active process interprets shell syntax. Forged bytes also advance one endpoint's sequence space, so continued control requires suppressing or rewriting competing traffic, translating sequence numbers, or accepting desynchronization. A saved packet capture cannot simply be replayed as a fresh authenticated connection because a new connection has different TCP state and must authenticate independently.

```bash
# live hijack: on-path, read seq/ack/window state and inject protocol-appropriate input
# in rlogin or an existing interactive shell, that input may be a shell command;
# in a fixed rsh/rexec command, it is only stdin for the already-running process
# classic tools (Hunt, Juggernaut) automated rsh/rlogin TCP hijacking; modern
# exploitation crafts the injection segments directly from the observed stream.
```

## Exploitation notes

- Live hijacking can supply input as the authenticated user, but command execution depends on the active process. Planting `.rhosts` directly requires an interactive shell or another command interpreter.
- A single injection is not inherently quiet: unless subsequent traffic is adjusted or suppressed, forged bytes desynchronize the legitimate endpoint and can expose the attack.
- It requires an active on-path position, visibility into the current stream state, and the ability to inject packets; a passive capture obtained after the connection closes is insufficient.
- It is the active counterpart to [command interception](command-interception.md) and the same class of technique as [rlogin session hijacking](../rlogin/session-hijacking.md) and classic Telnet hijacking.

## References

- [HackTricks: r-commands hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
