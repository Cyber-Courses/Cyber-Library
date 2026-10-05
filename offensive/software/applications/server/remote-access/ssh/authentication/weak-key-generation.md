---
title: "Weak key generation: predictable SSH keys from broken generators"
description: "SSH keys generated with insufficient entropy or by flawed tooling are predictable or factorable, so the private key can be derived without stealing it. The Debian OpenSSL entropy flaw produced a small, enumerable keyspace whose keys are brute-forced against authorized_keys, and low-entropy embedded devices and shared-factory keys repeat keys across units."
keywords:
  - weak key
  - debian openssl
  - predictable key
  - low entropy
  - factored key
---

# Weak key generation

Public-key authentication assumes the private key cannot be derived, but keys generated badly break that assumption. The archetype is the Debian OpenSSL entropy flaw: for a period, keys generated on affected systems drew from a tiny entropy pool, so only a few hundred thousand distinct keys per type/size existed, and the entire set was precomputed. Any public key from an affected system is matched to its private key by table lookup. Beyond that, embedded and IoT devices with poor entropy at first boot generate predictable keys, and many ship identical factory keys across all units, so recovering one device's key opens every peer.

```bash
# identify the key type/size in use (from a captured public key or host key)
ssh-keygen -lf authorized_keys                 # fingerprint and bit length
# Debian weak keys: match a target public key against the precomputed set
#   download the known weak-key set for the type/size, grep the target's blob
grep -f <(cut -d' ' -f2 target.pub) debian_ssh_rsa_2048/*.pub   # find the matching private key
# then authenticate with the recovered private key
ssh -i debian_ssh_rsa_2048/<match> user@<target>
# embedded/shared factory keys: extract from one device's firmware, reuse on peers
```

## Exploitation notes

- The Debian weak-key case is a lookup, not a crack: precomputed key sets exist, so a public key (host key or an `authorized_keys` entry) from an affected system directly yields the private key.
- Short or legacy keys (512/1024-bit RSA, DSA) are additionally weak to factoring or inherent flaws; `ssh-keygen -lf` reveals the size to judge this.
- Embedded devices often reuse one factory key across an entire product line; extracting it from firmware (or one unit) authenticates to all of them, and the same key frequently serves as both host key and a trusted client key.
- This complements [Public key exposure](public-key-exposure.md): there you steal the key, here you derive or reuse it; both end in authenticating as the owner.

## Tools

- [g0tmi1k debian-ssh weak key sets](https://github.com/g0tmi1k/debian-ssh)
- [ssh-keygen](https://man.openbsd.org/ssh-keygen)

## References

- [Debian OpenSSL predictable PRNG advisory](https://www.debian.org/security/2008/dsa-1571)
- [HackTricks: SSH weak keys](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
