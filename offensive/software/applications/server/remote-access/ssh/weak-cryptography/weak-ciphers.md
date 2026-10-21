---
title: "Weak ciphers: legacy and CBC-mode SSH encryption weaknesses"
order: 3
description: "Identifying legacy SSH ciphers and scoping their real impact, including the narrow plaintext-recovery conditions of the SSH CBC attack."
keywords:
  - weak cipher
  - cbc mode
  - rc4 arcfour
  - 3des
  - ssh encryption
---

# Weak ciphers

SSH encrypts the session with a negotiated cipher, and old servers may still offer legacy choices. CBC-mode ciphers (for example `aes128-cbc` and `3des-cbc`) were subject to a probabilistic plaintext-recovery attack that exploited SSH packet-length handling. Its yield and preconditions are narrow; merely offering CBC does not let an on-path attacker tamper with an authenticated session. The `arcfour` family uses RC4 and is obsolete, while 3DES has a small block size and limited security margin. Advertising these algorithms is evidence of legacy support, but impact depends on which cipher a client actually negotiates.

```bash
# which ciphers are offered?
nmap -p22 --script ssh2-enum-algos <target> | grep -A20 encryption_algorithms
# request a legacy/CBC cipher to confirm acceptance
ssh -c aes128-cbc user@<target>
ssh -c 3des-cbc user@<target>
ssh -c arcfour user@<target>        # RC4, if offered
```

## Exploitation notes

- The offered `encryption_algorithms` list identifies legacy support. Capture the negotiated cipher separately before assessing a session.
- The CBC plaintext-recovery result is probabilistic and limited. Do not report arbitrary decryption, key recovery, or packet tampering from CBC support alone.
- Requesting a cipher with `ssh -c` confirms that the server accepts it when a client explicitly selects it; this is not an active downgrade of another client's connection.
- Legacy ciphers often accompany old MAC and KEX options, but each category needs its own negotiated-algorithm and attack-precondition analysis. See [weak MAC algorithms](weak-mac-algorithms.md) and [legacy key exchange](key-exchange-downgrade.md).

## References

- [SSH CBC plaintext recovery (CPNI advisory)](https://www.openssh.com/txt/cbc.adv)
- [RFC 4253: encryption](https://datatracker.ietf.org/doc/html/rfc4253)
