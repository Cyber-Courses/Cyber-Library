---
title: "Tunneling: pivoting through SSH"
description: "SSH's forwarding features turn one reachable host into a network pivot: local and remote port forwarding expose services across the network boundary, dynamic forwarding provides a SOCKS proxy for arbitrary onward access, jump-host chaining reaches deep into segmented networks, and a forwarded agent on a compromised host is reusable to move further."
keywords:
  - ssh tunneling
  - port forwarding
  - socks proxy
  - proxyjump
  - agent forwarding
---

# Tunneling

SSH is not only a shell; its forwarding features make it a pivoting tool, and an attacker uses them exactly as an administrator would. Local and remote port forwarding bridge a service across a network boundary the attacker cannot otherwise cross. Dynamic forwarding stands up a SOCKS proxy that routes arbitrary tools through the SSH host into networks behind it. Jump-host (ProxyJump) chaining hops through bastions deep into segmented environments. And SSH agent forwarding, when a user forwards their agent to a host the attacker controls, leaves a usable authentication channel to move onward as that user.

```bash
# the four building blocks
ssh -L 8080:internal:80 user@pivot             # local forward: reach internal:80 via pivot
ssh -R 9001:localhost:9001 user@pivot          # remote forward: expose attacker service on pivot
ssh -D 1080 user@pivot                          # dynamic: SOCKS proxy through pivot
ssh -J user@bastion user@internal               # jump through bastion to internal
```

## Subtopics

- **[Port forwarding](port-forwarding.md)**: local and remote forwarding across boundaries.
- **[SOCKS proxy](socks-proxy.md)**: dynamic forwarding for arbitrary onward access.
- **[Jump host abuse](jump-host-abuse.md)**: chaining through bastions into segmented networks.
- **[Agent hijacking](agent-hijacking.md)**: reusing a forwarded SSH agent.

## References

- [OpenSSH port forwarding](https://man.openbsd.org/ssh#L)
- [HackTricks: SSH tunneling](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
