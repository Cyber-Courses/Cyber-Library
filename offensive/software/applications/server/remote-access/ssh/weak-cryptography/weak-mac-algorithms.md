---
title: "Weak MAC algorithms: broken SSH integrity protection"
description: "SSH protects each packet with a message authentication code, and servers that offer weak MACs, MD5-based, 96-bit truncated, or non-encrypt-then-MAC modes, weaken the integrity guarantee. A positioned attacker exploits weak or truncated MACs to tamper with or forge packets, undermining the protection that prevents session manipulation."
keywords:
  - weak mac
  - hmac-md5
  - truncated mac
  - encrypt-then-mac
  - integrity
---

# Weak MAC algorithms

SSH appends a message authentication code (MAC) to every packet so tampering is detected. The strength of that guarantee depends on the negotiated MAC, and old servers offer weak ones: MD5-based MACs (`hmac-md5`), 96-bit truncated MACs (`hmac-*-96`), and MACs used in the weaker encrypt-and-MAC (rather than encrypt-then-MAC, `*-etm`) construction. Weak or truncated MACs reduce the work to forge or tamper with packets, so a positioned attacker uses them to manipulate the session, undermining the integrity protection that otherwise blocks injection and modification attacks.

```bash
# which MACs are offered?
nmap -p22 --script ssh2-enum-algos <target> | grep -A20 mac_algorithms
# force a weak MAC to confirm acceptance
ssh -o MACs=hmac-md5 user@<target>
ssh -o MACs=hmac-sha1-96 user@<target>        # 96-bit truncated
```

## Exploitation notes

- The offered `mac_algorithms` list flags the weakness: `hmac-md5*`, any `*-96` truncation, and the absence of `-etm` (encrypt-then-MAC) variants indicate weaker integrity.
- Truncated and MD5 MACs lower the effort to forge a valid tag for tampered ciphertext; combined with a weak CBC cipher and an on-path position, this enables session manipulation rather than only passive observation.
- Like the cipher and KEX weaknesses, weak MACs mainly matter to an attacker who can capture or relay the traffic; they also fingerprint an outdated SSH build.
- These three weak-crypto pages compound: a server offering weak MACs, CBC ciphers, and small KEX groups is a candidate for cryptographic session attacks and for SSH machine-in-the-middle, see [SSH MITM](../mitm-and-trust/ssh-mitm.md).

## References

- [RFC 4253: data integrity](https://datatracker.ietf.org/doc/html/rfc4253)
- [OpenSSH encrypt-then-MAC](https://www.openssh.com/txt/release-6.2)
