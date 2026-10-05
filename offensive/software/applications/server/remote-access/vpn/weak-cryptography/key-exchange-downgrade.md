---
title: "Key exchange downgrade: weak VPN key-exchange groups"
description: "VPNs negotiating weak key-exchange parameters, small Diffie-Hellman groups (768/1024-bit) or legacy IKE groups, let a positioned or well-resourced attacker attack the key agreement. Forcing or observing a weak group enables precomputation attacks against the shared parameters and decryption of the session keys derived from them."
keywords:
  - key exchange
  - diffie-hellman
  - dh group
  - logjam
  - downgrade
---

# Key exchange downgrade

A VPN derives its session keys through a key exchange, and weak parameters make that agreement attackable. IPsec/IKE negotiates a Diffie-Hellman group from the offered transforms, and gateways that still offer small groups (DH group 1 = 768-bit, group 2 = 1024-bit) are exposed: these groups are within reach of precomputation attacks against their fixed shared parameters (the Logjam class), after which individual key exchanges using them are broken relatively cheaply. A downgrade, forcing or the gateway simply accepting a small group, drops the session to this attackable level, and the derived session keys (and thus the tunnel) fall with it.

```bash
# which DH groups does the IKE gateway offer?
ike-scan -M <target>                            # transform output includes the DH group
#   group 1 (768) / group 2 (1024) => small, precomputation-vulnerable
# force a small group to confirm acceptance, or observe a handshake that used one;
# attacking the fixed 1024-bit group parameters (Logjam-style) then breaks the exchange
```

## Exploitation notes

- Offered DH groups 1 and 2 (768/1024-bit) are the finding; their fixed parameters are susceptible to a one-time precomputation that then cheaply breaks any exchange using them, so a gateway accepting them is attackable by a well-resourced adversary.
- Downgrade applies where the gateway accepts a weak group alongside strong ones; an attacker forces the weak selection or waits for a client that negotiates it.
- This attacks the key agreement itself (yielding the session keys), distinct from [weak ciphers](weak-ciphers.md) which attack the bulk encryption; both end in decryptable tunnel traffic.
- Pair with a capture position; the practical payoff is decrypting a captured tunnel that used the weak group.

## References

- [Logjam / weak Diffie-Hellman](https://weakdh.org/)
- [NIST SP 800-77: IKE groups](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
