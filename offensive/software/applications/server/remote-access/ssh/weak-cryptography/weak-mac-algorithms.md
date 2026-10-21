---
title: "Weak SSH MAC algorithms: legacy integrity protection"
order: 2
description: "Identifying legacy SSH MAC algorithms, understanding what their negotiation does and does not prove, and distinguishing cryptographic deprecation from a practical session attack."
keywords:
  - weak mac
  - hmac-md5
  - truncated mac
  - encrypt-then-mac
  - integrity
---

# Weak MAC algorithms

SSH appends a message authentication code (MAC) to each protected packet so tampering is detected. Old servers may still offer MD5-based MACs (`hmac-md5`), 96-bit truncated MACs (`hmac-*-96`), or the original encrypt-and-MAC construction instead of encrypt-then-MAC (`*-etm`). These choices are legacy, but negotiation alone is not proof that an attacker can forge packets or manipulate a session.

A valid 96-bit tag still requires an infeasible online guessing effort in the absence of a separate weakness, and collision attacks against bare MD5 do not directly become HMAC forgeries. The practical finding is obsolete cryptographic configuration, reduced security margin, and exposure to protocol or implementation attacks that may depend on the original MAC construction. A session-compromise claim needs a named attack with matching cipher, MAC, implementation, and network-position preconditions.

```bash
# which MACs are offered?
nmap -p22 --script ssh2-enum-algos <target> | grep -A20 mac_algorithms
# force a weak MAC to confirm acceptance
ssh -o MACs=hmac-md5 user@<target>
ssh -o MACs=hmac-sha1-96 user@<target>        # 96-bit truncated
```

## Exploitation notes

- Treat `hmac-md5*`, `hmac-*-96`, and a lack of `*-etm` options as legacy-configuration evidence, not automatic exploitation evidence.
- Confirm which MAC the client and server actually negotiate. An offered legacy algorithm is lower severity when both peers select a modern alternative.
- Record the cipher, key exchange, implementation, and version before assessing impact. Some attacks depend on a particular combination rather than the MAC in isolation.
- An on-path attacker still has to defeat SSH host authentication or exploit a separate protocol or implementation flaw. Merely relaying traffic does not reveal the session keys or permit a valid MAC forgery.

## References

- [RFC 4253: data integrity](https://datatracker.ietf.org/doc/html/rfc4253)
- [OpenSSH encrypt-then-MAC](https://www.openssh.com/txt/release-6.2)
