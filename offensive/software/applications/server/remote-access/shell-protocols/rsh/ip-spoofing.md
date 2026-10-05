---
title: "IP spoofing: impersonating a trusted host to rsh"
description: "The r-commands authenticate by source IP against the trust files, so spoofing the address of a trusted host can satisfy the check without controlling it. Because rsh/rlogin need the server's responses, the attack requires either an on-path position or predicting TCP sequence numbers, the classic blind-spoofing attack against address-based trust."
keywords:
  - ip spoofing
  - trusted host
  - tcp sequence prediction
  - blind spoofing
  - rsh
---

# IP spoofing

The r-commands decide trust from the client's source IP (resolved against `.rhosts`/`hosts.equiv`), so an attacker who makes their packets appear to come from a trusted host satisfies the check without owning that host. The complication is that rsh/rlogin are TCP and interactive, so the attacker normally needs the server's replies. Two routes solve this: an on-path position (seeing the responses directly), or blind spoofing, predicting the server's TCP initial sequence numbers to complete the handshake and inject the command without seeing the replies, the classic attack that address-based trust invites.

```bash
# on-path: spoof the trusted source and relay/observe responses, then:
rsh -l root <target> 'echo "+ +" >> /root/.rhosts'    # as the trusted host, open a backdoor
# blind spoofing (historic): predict the target's TCP ISN to forge a full connection
#   from the trusted host's address and inject a command (e.g. writing .rhosts),
#   converting a one-shot spoof into durable passwordless access
```

## Exploitation notes

- The goal is usually not a live session (hard when blind) but injecting one command that establishes durable trust, classically `echo "+ +" >> ~/.rhosts` for the target user, after which the attacker connects normally.
- On-path spoofing is straightforward where you can see the trusted host's traffic; blind spoofing depends on predictable TCP sequence numbers, which modern stacks randomise, so it is mainly viable against old systems (exactly where r-commands live).
- This is the attack that address-based trust fundamentally enables; it needs no credential and no write access, only the ability to forge a trusted source and get one command executed.
- Combine with knowledge of which hosts are trusted (from a readable [hosts.equiv](hosts-equiv.md) or `.rhosts`) to pick a spoofable trusted address.

## References

- [Morris/Bellovin TCP sequence prediction](https://www.rfc-editor.org/rfc/rfc1948)
- [HackTricks: rsh trust](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
