---
title: "Key exchange downgrade: weak VPN key-exchange groups"
description: "VPNs that negotiate small Diffie-Hellman groups, such as legacy IKE groups 1 and 2, use key-exchange parameters below current recommendations. A well-resourced attacker may benefit from precomputation against widely reused finite-field groups, but offering a group alone does not prove that an attacker can force its selection."
keywords:
  - key exchange
  - diffie-hellman
  - dh group
  - logjam
  - downgrade
---

# Key exchange downgrade

A VPN derives its session keys through a key exchange, and undersized parameters reduce the security margin of that agreement. IPsec/IKE can negotiate finite-field Diffie-Hellman groups from the configured proposals; group 1 uses a 768-bit modulus and group 2 uses a 1024-bit modulus, both below current recommendations. Widely reused finite-field groups can make large precomputation efforts more valuable, but acceptance of a weak group is not equivalent to a practical session compromise. In IKEv2, the subsequent `IKE_AUTH` exchange authenticates the initial negotiation, so an unauthenticated attacker cannot simply rewrite the proposal and complete a silent downgrade.

```bash
# which DH groups does the IKE gateway offer?
ike-scan -M <target>                            # transform output includes the DH group
#   group 1 (768) / group 2 (1024) => small, precomputation-vulnerable
# confirm whether a configured client actually negotiates a legacy group
# distinguish an offered proposal from the transform selected in a completed exchange
```

## Exploitation notes

- Offering groups 1 or 2 is a configuration finding because these groups are below current recommendations; record separately whether a client actually selects one in a completed exchange.
- Do not report an active downgrade solely from the proposal list. IKEv2 authenticates the initial exchange during `IKE_AUTH`; forcing a different group requires a separate protocol or implementation weakness, or control of a client that willingly negotiates it.
- Precomputation against a reused finite-field group is a resource-intensive threat model, not a generic guarantee that every observed 1024-bit exchange can be cheaply broken.
- This concerns the key agreement, distinct from [weak ciphers](weak-ciphers.md), which govern bulk encryption. State the adversary position, resources, negotiated group, and evidence when assessing practical impact.

## References

- [Logjam / weak Diffie-Hellman](https://weakdh.org/)
- [NIST SP 800-77: IKE groups](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [RFC 7296: IKEv2](https://www.rfc-editor.org/rfc/rfc7296.html)
