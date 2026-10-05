---
title: "Weak pre-shared keys: cracking VPN PSKs"
description: "IPsec VPNs using a pre-shared key for authentication are vulnerable when that key is weak or default, especially in IKE aggressive mode, which sends a hash of the PSK that an attacker captures and cracks offline. Recovering the PSK, combined with any required XAUTH credentials, authenticates the attacker to the VPN."
keywords:
  - pre-shared key
  - psk
  - ike aggressive mode
  - offline cracking
  - psk-crack
---

# Weak pre-shared keys

IPsec (and L2TP/IPsec) VPNs often authenticate the tunnel with a pre-shared key, and a weak or default PSK is recoverable. The sharpest case is IKE aggressive mode: to save a round trip, aggressive mode sends a hash computed from the PSK in the first exchange, before the key is confirmed, so an attacker who initiates an aggressive-mode handshake captures that hash and cracks the PSK offline with a wordlist or brute force. Once the PSK is recovered, the attacker completes the Phase 1 authentication (and then any XAUTH username/password, itself often weak), establishing the tunnel.

```bash
# detect and capture an aggressive-mode PSK hash
ike-scan -A -M -P psk.hash <target>             # -A aggressive, -P saves the hash
# crack the captured PSK offline
psk-crack -d wordlist.txt psk.hash
psk-crack -b 8 psk.hash                          # brute force up to length 8
# with the PSK (and XAUTH creds if required), connect
```

## Exploitation notes

- Aggressive mode is the enabler: it emits the PSK-derived hash to an unauthenticated initiator, so `ike-scan -A -P` captures it and `psk-crack` recovers a weak key offline; main mode does not leak this, so check which modes the gateway offers.
- Default PSKs (vendor defaults, documentation examples, obvious strings) and short keys fall quickly; seed the wordlist with organisation-specific terms.
- The PSK authenticates the tunnel but many deployments add XAUTH (a username/password) as a second factor; recover the PSK first, then attack XAUTH ([password brute force](password-brute-force.md)).
- A recovered PSK plus XAUTH yields a full tunnel into the internal network; see [IPsec IKE](../protocols/ipsec-ike.md) for the aggressive-mode mechanics.

## Tools

- [ike-scan / psk-crack](https://github.com/royhills/ike-scan)

## References

- [HackTricks: IKE aggressive mode PSK](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
- [NIST SP 800-77: IKE](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
