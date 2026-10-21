---
title: "Weak key generation: predictable SSH keys from broken generators"
description: "Flawed random-number generation can create predictable SSH keys, as in CVE-2008-0166 on affected Debian-derived systems. Distinguish user keys, which authorize clients, from host keys, which identify servers, and distinguish obsolete key sizes from keys that are practically factorable."
keywords:
  - weak key
  - debian openssl
  - predictable key
  - low entropy
  - factored key
---

# Weak key generation

CVE-2008-0166 was caused by a Debian-specific change that made OpenSSL-generated key material predictable in affected releases. Debian required regeneration of both weak OpenSSH user keys and host keys and added blacklist detection. The consequences differ: a recovered user private key may authenticate to accounts that trust it, while a recovered host private key can support server impersonation or attacks on host identity. Other low-entropy or duplicated-device-key findings require product- and firmware-specific evidence.

```bash
# identify key type and size
ssh-keygen -lf authorized_keys                 # fingerprint and bit length
# on systems retaining Debian's blacklist utility, check keys against known weak sets
ssh-vulnkey target.pub
# interpret a match according to whether target.pub is a user or host key
```

## Exploitation notes

- Known CVE-2008-0166 public keys can be compared with precomputed weak-key sets; impact then depends on how that key is used and trusted.
- RSA keys below 2048 bits are obsolete under current security-strength guidance, but that does not mean an arbitrary 1024-bit RSA modulus is presently practical to factor.
- DSA has separate algorithm and parameter concerns; identify the exact problem instead of treating every legacy DSA key as factorable.
- Repeated embedded-device keys are a product-specific finding. A duplicated host key affects server identity; a duplicated authorized client key can affect login authorization.

## Tools

- [g0tmi1k debian-ssh weak key sets](https://github.com/g0tmi1k/debian-ssh)
- [ssh-keygen](https://man.openbsd.org/ssh-keygen)

## References

- [Debian OpenSSL predictable PRNG advisory](https://www.debian.org/security/2008/dsa-1571)
- [Debian OpenSSH advisory](https://lists.debian.org/debian-security-announce/2008/msg00153.html)
- [NIST cryptographic algorithm transition guidance](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/validation-notes)
