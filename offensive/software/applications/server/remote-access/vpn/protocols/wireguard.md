---
title: "WireGuard: static-key management and identity exposure"
order: 4
description: "WireGuard's cryptography is modern and sound, so attacks target its key management: recovering the static private keys and pre-shared keys from client and server configuration files, where they sit in cleartext, and the identity exposure from its fixed per-peer key model. A recovered key authenticates the attacker as that peer."
keywords:
  - wireguard
  - static key
  - pre-shared key
  - wg0.conf
  - key management
---

# WireGuard

WireGuard is deliberately minimal and cryptographically sound, so there is no protocol-level crypto attack of the kind that afflicts PPTP or aggressive-mode IKE; it even stays silent to unauthenticated probes. The attack surface is therefore key management. WireGuard authenticates peers with static Curve25519 key pairs and an optional pre-shared key, and these live in cleartext in configuration files (`wg0.conf`, client `.conf` files) and in the running configuration. Recovering a peer's private key (and the PSK, if used) from a config, backup, or device lets the attacker stand up a peer with that identity and connect. The fixed-key model also means a peer's public key is a persistent identity that reveals who is connecting.

```bash
# recover keys from configuration (cleartext)
cat /etc/wireguard/wg0.conf                      # PrivateKey, PresharedKey, peer PublicKeys
grep -rE 'PrivateKey|PresharedKey' /etc/wireguard /mnt/loot ~/ 2>/dev/null
# the running config also exposes them
wg showconf wg0
# with a peer's PrivateKey (+ PSK if set), configure a client to connect as that peer
```

## Exploitation notes

- The whole attack is finding the static private key (and PSK): they are stored in cleartext in `wg0.conf`/client configs and visible via `wg showconf`, so a readable config, a backup, or a compromised endpoint yields a usable identity.
- With a peer's private key and the matching endpoint/allowed-IPs (also in the config), the attacker brings up a peer and connects as that identity; no cracking is involved because the protocol is sound.
- The optional pre-shared key adds a symmetric factor; it too lives in the config, so recovering the config usually recovers both.
- Protect-the-config is the real control; treat WireGuard key material like any high-value private key and hunt for it accordingly, mirroring [certificate abuse](../authentication/certificate-abuse.md).

## References

- [WireGuard protocol and configuration](https://www.wireguard.com/)
- [wg(8) and wg-quick](https://man7.org/linux/man-pages/man8/wg.8.html)
