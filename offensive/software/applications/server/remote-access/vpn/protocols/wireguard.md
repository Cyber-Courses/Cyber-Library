---
title: "WireGuard: static-key management and identity considerations"
order: 4
description: "WireGuard attacks primarily target static private keys and optional pre-shared keys stored in configuration or active interface state. The handshake hides static public-key identity from passive observers unless relevant long-term key material is later compromised."
keywords:
  - wireguard
  - static key
  - pre-shared key
  - wg0.conf
  - key management
---

# WireGuard

WireGuard is deliberately minimal and uses modern cryptography, so the primary attack surface is key management. WireGuard authenticates peers with static Curve25519 key pairs and an optional pre-shared key; deployments commonly store these values in configuration files (`wg0.conf`, client `.conf` files) and active interface state. Recovering a peer's private key, plus its pre-shared key when configured, can let an attacker impersonate that peer. The fixed public key is a persistent configured identity, but WireGuard encrypts the initiator's static public key during the handshake, so a passive observer does not ordinarily learn it. If the responder's private key is later compromised, recorded historical handshakes can be associated with initiator identities, although past tunnel data retains forward secrecy.

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
- [WireGuard known limitations: identity-hiding forward secrecy](https://www.wireguard.com/known-limitations/)
- [wg(8) and wg-quick](https://man7.org/linux/man-pages/man8/wg.8.html)
