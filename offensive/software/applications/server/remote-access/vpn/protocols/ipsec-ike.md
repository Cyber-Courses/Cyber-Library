---
title: "IPsec IKE: aggressive-mode PSK disclosure and IKE weaknesses"
order: 1
description: "IPsec key exchange via IKE exposes several attacks: aggressive mode discloses a hash of the pre-shared key to an unauthenticated initiator for offline cracking, the gateway can be fingerprinted and enumerated through IKE, and weak transforms (small DH groups, DES, SHA-1) negotiated in Phase 1 weaken the tunnel. XAUTH, where added, is a further credential target."
keywords:
  - ipsec
  - ike
  - aggressive mode
  - pre-shared key
  - xauth
---

# IPsec IKE

IPsec uses IKE (on UDP 500, with NAT-traversal on 4500) to authenticate peers and establish keys, and IKE is the main attack surface. The signature weakness is aggressive mode: to complete Phase 1 in fewer messages, it sends a hash derived from the pre-shared key in the clear to whoever initiates, so an unauthenticated attacker captures that hash and cracks the PSK offline. IKE also fingerprints the gateway (vendor ID, transform sets), reveals weak crypto support, and, where XAUTH adds a username/password on top of the PSK, presents a second credential to attack. Main mode does not leak the PSK hash, so the gateway's mode support matters.

```bash
# fingerprint and detect aggressive mode
ike-scan -M <target>                            # main-mode transforms + vendor ID
ike-scan -A -M -P psk.hash <target>             # aggressive mode; -P captures the PSK hash
# crack the captured PSK offline
psk-crack -d wordlist.txt psk.hash
# with the PSK, attack XAUTH credentials if required, then establish the tunnel
```

## Exploitation notes

- Aggressive mode is the key finding: `ike-scan -A -P` captures the PSK-derived hash from an unauthenticated handshake, and `psk-crack` recovers a weak key offline, see [weak pre-shared keys](../authentication/weak-pre-shared-keys.md).
- The IKE fingerprint (vendor ID, transforms) identifies the device and its weak-crypto support, feeding [key-exchange downgrade](../weak-cryptography/key-exchange-downgrade.md) and device-specific exploits.
- XAUTH adds a username/password after PSK authentication; recover the PSK, then spray XAUTH credentials ([password brute force](../authentication/password-brute-force.md)).
- Group-ID/user enumeration is possible on some gateways via IKE responses; combine with the recovered PSK to complete the tunnel into the internal network.

## Tools

- [ike-scan / psk-crack](https://github.com/royhills/ike-scan)

## References

- [HackTricks: IPsec/IKE aggressive mode](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
- [RFC 2409 (IKE)](https://datatracker.ietf.org/doc/html/rfc2409)
