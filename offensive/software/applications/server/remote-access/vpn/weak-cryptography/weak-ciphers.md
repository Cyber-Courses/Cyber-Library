---
title: "Weak ciphers: legacy and broken VPN cipher negotiation"
description: "VPN gateways that permit legacy ciphers, single or triple DES, RC4, or export-grade suites can establish sessions with obsolete protection. Offered and negotiated algorithms should be distinguished, and practical impact depends on the cipher and attack conditions."
keywords:
  - weak cipher
  - 3des
  - rc4
  - transform set
  - decryption
---

# Weak ciphers

A VPN that still offers weak encryption can establish sessions with obsolete protection. In IPsec the gateway advertises transform sets, while TLS-based VPNs advertise cipher suites. DES, 3DES, RC4, export-grade suites, and other legacy choices should be removed, but an offered algorithm is not necessarily the one a client negotiates and does not by itself demonstrate a forced downgrade. Confirm the selected algorithm and describe its specific exposure: for example, SWEET32-style attacks against 64-bit block ciphers require a large volume of traffic under the same keys, whereas export-grade and RC4 attacks have different prerequisites.

```bash
# IPsec: which transforms are offered (look for DES/3DES, weak hashes)
ike-scan -M <target>
# TLS VPN: grade the cipher suites
nmap -p443 --script ssl-enum-ciphers <target>        # flags RC4, 3DES, export, SWEET32-prone
# capture tunnel traffic for offline analysis where a weak suite is in use
```

## Exploitation notes

- Offered transform sets and cipher suites establish server support; capture a completed negotiation to show what a representative client actually selects.
- 64-bit block ciphers such as 3DES are vulnerable to birthday-bound collision attacks on long-lived, high-volume sessions (the SWEET32 class); their presence does not make an arbitrary short capture immediately decryptable.
- Protocol negotiation is authenticated when the handshake completes successfully. Claim a forced downgrade only when a separate fallback, protocol, or implementation weakness supports it.
- Even where decryption is impractical, weak-cipher support fingerprints an old gateway likely vulnerable to the [appliance](../ssl-vpn-appliances/index.md) or protocol flaws.

## References

- [SWEET32 (64-bit block cipher attacks)](https://sweet32.info/)
- [nmap ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)
