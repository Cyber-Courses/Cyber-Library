---
title: "IP spoofing: impersonating a trusted source for the r-commands"
order: 1
description: "Because r-command trust is checked against the source IP, spoofing the address of a trusted host satisfies the check without controlling it. With an on-path position the attacker sees responses directly; blind, they predict TCP sequence numbers to complete the connection and inject a command, the classic attack against address-based trust."
keywords:
  - ip spoofing
  - source address
  - tcp sequence prediction
  - trusted host
  - blind spoofing
---

# IP spoofing

Host-based trust is evaluated from the connection's source IP, so forging that address to match a trusted host defeats the check without owning the host. Across all r-commands this is the attack that address-based authentication invites. With an on-path position the attacker spoofs the trusted source and observes the server's replies, completing an interactive exchange. Blind, without seeing replies, the attacker predicts the server's TCP initial sequence numbers to forge a complete connection from the trusted address and inject a single command, classically one that establishes durable trust so later access needs no spoofing.

```bash
# identify a trusted host to impersonate (from a readable hosts.equiv/.rhosts)
cat /etc/hosts.equiv ~*/.rhosts 2>/dev/null
# on-path: spoof the trusted source, relay/observe responses, then run a command
rsh -l root <target> 'echo "+ +" >> /root/.rhosts'    # as the trusted host
# blind: predict the target TCP ISN, forge the connection from the trusted address,
#   and inject one command (e.g. writing .rhosts) to convert the spoof into durable access
```

## Exploitation notes

- The usual objective is to inject one command that plants durable trust ([rhosts write](rhosts-write.md)), after which the attacker connects normally without spoofing.
- On-path spoofing is easy where you see the trusted host's traffic; blind spoofing needs predictable TCP sequence numbers, which modern stacks randomise, so it is viable mainly against the old systems r-commands run on.
- Needs no credential and no write access, only a forged trusted source and one executed command; it is the purest expression of the address-based-trust weakness.
- The specific trusted address to forge comes from reading `hosts.equiv`/`.rhosts` or inferring the trust relationships.

## References

- [RFC 1948 (defending against sequence prediction)](https://www.rfc-editor.org/rfc/rfc1948)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
