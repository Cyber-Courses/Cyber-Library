---
title: "IP spoofing: impersonating a trusted host to rsh"
order: 2
description: "Traditional rsh trust considers a privileged client source port, remote hostname or address, remote username, requested local account, and the matching trust rule. An on-path attacker may impersonate a trusted endpoint; blind spoofing additionally requires predictable TCP sequence state and protocol-correct packet injection."
keywords:
  - ip spoofing
  - trusted host
  - tcp sequence prediction
  - blind spoofing
  - rsh
---

# IP spoofing

Traditional `rshd` does not decide trust from the source IP alone. It expects the client to use a privileged source port, may validate the source address through hostname resolution, retrieves both remote and local usernames, and checks the resulting identity against `.rhosts` or `hosts.equiv`. An attacker must satisfy that complete decision while impersonating a trusted endpoint. An on-path position provides visibility into the TCP exchange; blind spoofing additionally requires predicting TCP sequence state and forging the handshake and null-terminated rsh protocol fields without seeing replies.

```bash
# determine the trusted host, remote user, requested local account, source-port
# requirement, hostname checks, and exact trust rule used by the target
# on-path: intercept or forge a protocol-correct exchange while observing TCP state
# blind: predict TCP state and inject the complete null-terminated rsh request fields
# a successful request might plant .rhosts to convert temporary access into persistence
```

## Exploitation notes

- The goal is usually not a live session (hard when blind) but injecting one command that establishes durable trust, classically `echo "+ +" >> ~/.rhosts` for the target user, after which the attacker connects normally.
- An on-path position exposes TCP state, but the attacker must still satisfy the privileged-port, hostname, user, target-account, and trust-rule checks. Blind spoofing additionally depends on predictable TCP sequence numbers, which modern stacks randomise, so it is mainly viable against old systems.
- The attack may need no password or prior file write, but it does require a matching trusted host/user identity and a protocol-correct connection.
- Combine with knowledge of which hosts are trusted (from a readable [hosts.equiv](hosts-equiv.md) or `.rhosts`) to pick a spoofable trusted address.

## References

- [Morris/Bellovin TCP sequence prediction](https://www.rfc-editor.org/rfc/rfc1948)
- [OpenBSD rshd protocol and authentication checks](https://man.openbsd.org/OpenBSD-5.3/rshd.8)
- [HackTricks: rsh trust](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
