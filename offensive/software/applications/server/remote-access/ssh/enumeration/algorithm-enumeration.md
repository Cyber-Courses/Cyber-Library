---
title: "Algorithm enumeration: detecting SSH weak-crypto exposure"
description: "An SSH server advertises its supported key-exchange, host-key, cipher, and MAC algorithms during the unauthenticated handshake. Enumerating and grading them reveals whether the server offers weak or legacy options, which both fingerprints an outdated build and identifies the concrete downgrade and weak-crypto attacks available against it."
keywords:
  - algorithm enumeration
  - ssh2-enum-algos
  - kex ciphers macs
  - weak crypto
  - handshake
---

# Algorithm enumeration

Part of the SSH handshake, sent before authentication, is each side's list of supported algorithms for key exchange, host-key signature, encryption, and MAC. Reading the server's lists tells you exactly what it will accept, and grading them against modern baselines identifies weak or legacy options. This serves two purposes: it fingerprints the age and configuration of the server (old builds offer broad legacy sets), and it enumerates the specific weak-crypto attacks available, since you cannot force an algorithm the server does not offer.

```bash
# enumerate the four algorithm lists the server offers
nmap -p22 --script ssh2-enum-algos <target>
# interpret: look for weak entries in each list
#   kex:     diffie-hellman-group1-sha1, *-sha1, small GEX groups
#   cipher:  *-cbc, arcfour*, 3des, des
#   mac:     hmac-md5*, *-96 (truncated), non -etm
#   hostkey: ssh-rsa (SHA-1) only, ssh-dss (DSA)
ssh -vv user@<target> 2>&1 | grep -iE 'kex:|cipher:|mac:'   # what gets negotiated
```

## Exploitation notes

- Map each offered list to its weakness: `group1`/SHA-1 KEX to [key exchange downgrade](../weak-cryptography/key-exchange-downgrade.md), `-cbc`/`arcfour`/`3des` to [weak ciphers](../weak-cryptography/weak-ciphers.md), `hmac-md5`/`-96` to [weak MAC algorithms](../weak-cryptography/weak-mac-algorithms.md).
- `ssh-dss` (DSA) host keys and SHA-1-only `ssh-rsa` indicate an old server; DSA is disabled in modern OpenSSH, so its presence is a strong age signal.
- You can only attack algorithms the server offers, so this enumeration is the prerequisite that turns the weak-crypto pages from theory into a concrete target list.
- A fully modern algorithm set (curve25519 KEX, ChaCha20/AES-GCM ciphers, ETM MACs) means the crypto surface is closed; pivot to authentication and implementation vulnerabilities instead.

## Tools

- [nmap ssh2-enum-algos](https://nmap.org/nsedoc/scripts/ssh2-enum-algos.html)
- [ssh-audit](https://github.com/jtesta/ssh-audit)

## References

- [RFC 4253: algorithm negotiation](https://datatracker.ietf.org/doc/html/rfc4253)
- [ssh-audit hardening guides](https://www.ssh-audit.com/)
