---
title: "Weak ciphers: legacy and CBC-mode SSH encryption weaknesses"
order: 3
description: "SSH servers that still offer legacy ciphers, single-DES, RC4/arcfour, or CBC-mode ciphers, expose the session to known weaknesses. CBC mode in SSH was subject to a plaintext-recovery attack, and the stream and legacy ciphers are cryptographically weak, so a server advertising them lets a positioned attacker weaken or tamper with the encrypted channel."
keywords:
  - weak cipher
  - cbc mode
  - rc4 arcfour
  - 3des
  - ssh encryption
---

# Weak ciphers

SSH encrypts the session with a negotiated cipher, and old servers still offer weak ones. CBC-mode ciphers (for example `aes128-cbc`, `3des-cbc`) were subject to a classic SSH plaintext-recovery attack that exploited how CBC interacts with SSH's packet-length handling, letting an attacker recover limited plaintext from an observed session. The stream cipher `arcfour` (RC4) is cryptographically broken, and single-DES is trivially weak. A server advertising any of these indicates a weak configuration and, for an attacker who can capture or sit on the connection, a path to weakening or tampering with the channel.

```bash
# which ciphers are offered?
nmap -p22 --script ssh2-enum-algos <target> | grep -A20 encryption_algorithms
# force a weak/CBC cipher to confirm acceptance
ssh -c aes128-cbc user@<target>
ssh -c 3des-cbc user@<target>
ssh -c arcfour user@<target>        # RC4, if offered
```

## Exploitation notes

- The offered `encryption_algorithms` list is the tell: any `-cbc` entry, `arcfour*`, `3des`, or `des` marks a weak server configuration.
- The CBC plaintext-recovery weakness requires an on-path or capture position and specific conditions; its practical yield is limited plaintext, but it signals a server worth attacking cryptographically rather than only by authentication.
- Weak ciphers rarely stand alone; a server offering them typically also offers weak MACs and KEX, compounding the exposure, see [weak MAC algorithms](weak-mac-algorithms.md) and [key exchange downgrade](key-exchange-downgrade.md).
- Even where exploitation is impractical, the presence of weak ciphers fingerprints an old, likely-vulnerable SSH build worth checking for implementation CVEs.

## References

- [SSH CBC plaintext recovery (CPNI advisory)](https://www.openssh.com/txt/cbc.adv)
- [RFC 4253: encryption](https://datatracker.ietf.org/doc/html/rfc4253)
