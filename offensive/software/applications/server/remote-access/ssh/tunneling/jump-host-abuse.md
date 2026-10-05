---
title: "Jump host abuse: chaining through bastions into segmented networks"
description: "SSH jump-host support (ProxyJump / -J) transparently chains connections through one or more intermediate hosts, which is how bastions provide access to segmented networks. An attacker with access to or through a bastion uses the same mechanism to reach otherwise-isolated internal hosts, and a compromised bastion exposes every network it bridges."
keywords:
  - proxyjump
  - jump host
  - bastion
  - -J
  - multi-hop
---

# Jump host abuse

Organizations front segmented networks with bastion (jump) hosts: the only way into the protected segment is to SSH through the bastion. SSH's `ProxyJump` (`-J`) automates that hop, chaining the connection through one or more intermediaries transparently. An attacker uses the identical mechanism offensively: given access to the bastion (credentials, a key) or the ability to authenticate through it, they reach the internal hosts it bridges, and a fully compromised bastion is a gateway to every network segment it connects, plus a trove of the credentials and keys that transit it.

```bash
# chain through a bastion to an internal host in one command
ssh -J user@bastion user@10.0.9.15
# multi-hop through two jumps
ssh -J user@bastion1,user@bastion2 user@deep-internal
# make it persistent in ~/.ssh/config for tooling to use the chain
#   Host internal
#     HostName 10.0.9.15
#     ProxyJump user@bastion
```

## Exploitation notes

- The bastion is the high-value target: it is reachable by design and bridges into the protected segment, so compromising it (via the SSH authentication attacks) opens everything behind it.
- A bastion sees the credentials and keys of everyone who jumps through it; a foothold there enables capturing or reusing those (and agent forwarding through it is directly abusable, see [Agent hijacking](agent-hijacking.md)).
- `ProxyJump` chains cleanly for multi-layer environments; combine with a [SOCKS proxy](socks-proxy.md) on the final hop to run arbitrary tools against the deepest segment.
- Jump-host access plus a forwarded agent is a classic escalation: the bastion uses the jumping user's agent to authenticate onward, so the attacker on the bastion authenticates as that user to the next hop.

## References

- [OpenSSH: ProxyJump](https://man.openbsd.org/ssh_config#ProxyJump)
- [HackTricks: jump hosts](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
