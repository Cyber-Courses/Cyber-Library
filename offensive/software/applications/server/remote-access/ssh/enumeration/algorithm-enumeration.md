---
title: "Algorithm enumeration: detecting SSH weak-crypto exposure"
description: "An SSH server advertises key-exchange, host-key, cipher, and MAC algorithms before authentication. Offered legacy choices are configuration findings; practical impact depends on client overlap, the negotiated result, attacker position, and the prerequisites of a named weakness."
keywords:
  - algorithm enumeration
  - ssh2-enum-algos
  - kex ciphers macs
  - weak crypto
  - handshake
---

# Algorithm enumeration

During the SSH handshake, each side advertises supported algorithms for key exchange, host-key signatures, encryption, and integrity. The server list identifies what it may accept and supports configuration review, but an offered legacy option does not prove exploitation. A connection uses the first mutually supported choice according to SSH negotiation rules. Assess client overlap, the algorithm actually negotiated, attacker position, key sizes and parameters, and the prerequisites and impact of a named cryptographic weakness.

```bash
# enumerate the four algorithm lists the server offers
nmap -p22 --script ssh2-enum-algos <target>
# identify legacy entries for policy review
#   kex:     diffie-hellman-group1-sha1, *-sha1, small GEX groups
#   cipher:  *-cbc, arcfour*, 3des, des
#   mac:     hmac-md5*, *-96 (truncated), non -etm
#   hostkey: ssh-rsa (SHA-1) only, ssh-dss (DSA)
ssh -vv user@<target> 2>&1 | grep -iE 'kex:|cipher:|mac:'   # what gets negotiated
```

## Exploitation notes

- Map each offered list to its legacy category: `group1`/SHA-1 KEX to [legacy key exchange](../weak-cryptography/key-exchange-downgrade.md), `-cbc`/`arcfour`/`3des` to [weak ciphers](../weak-cryptography/weak-ciphers.md), and `hmac-md5`/`-96` to [weak MAC algorithms](../weak-cryptography/weak-mac-algorithms.md). Then determine what a real client negotiates before assigning session impact.
- `ssh-dss` (DSA) host keys and SHA-1-only `ssh-rsa` indicate an old server; DSA is disabled in modern OpenSSH, so its presence is a strong age signal.
- Server support is only one prerequisite: the client must also accept the option, and many weaknesses require an on-path position or additional conditions.
- A modern advertised set reduces legacy-algorithm exposure but does not close every cryptographic risk; implementation defects, randomness, key handling, parameter validation, and host-key trust remain separate concerns.

## Tools

- [nmap ssh2-enum-algos](https://nmap.org/nsedoc/scripts/ssh2-enum-algos.html)
- [ssh-audit](https://github.com/jtesta/ssh-audit)

## References

- [RFC 4253: algorithm negotiation](https://datatracker.ietf.org/doc/html/rfc4253)
- [OpenSSH legacy options](https://www.openssh.com/legacy.html)
