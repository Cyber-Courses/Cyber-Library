---
title: "Legacy SSH key exchange negotiation"
order: 1
description: "Identifying SSH servers that accept legacy key exchanges, distinguishing client-selected negotiation from an active downgrade, and scoping the risk of small Diffie-Hellman groups."
keywords:
  - key exchange
  - diffie-hellman
  - group1
  - downgrade
  - kex
---

# Legacy key exchange

The SSH key exchange (KEX) derives the symmetric session keys and authenticates the server's host key. Servers may still accept `diffie-hellman-group1-sha1` (a fixed 1024-bit group), SHA-1-based exchanges, or undersized group-exchange parameters for compatibility with old clients. A session is exposed to those weaknesses only when the client actually negotiates the legacy method.

SSH protects the negotiation against a transparent on-path downgrade. The server signs an exchange hash that includes both peers' complete `SSH_MSG_KEXINIT` payloads, so removing stronger algorithms from either list invalidates host authentication. An active attacker cannot simply rewrite the lists and force a weak KEX unless host-key verification is bypassed or a separate implementation flaw breaks that binding.

```bash
# which KEX methods does the server offer?
nmap -p22 --script ssh2-enum-algos <target> | grep -A15 kex_algorithms
# request a legacy KEX to confirm the server accepts it
ssh -o KexAlgorithms=diffie-hellman-group1-sha1 user@<target>
ssh -o KexAlgorithms=diffie-hellman-group-exchange-sha1 user@<target>
```

## Exploitation notes

- Enumerating `kex_algorithms` identifies legacy support. It does not show which method normal clients negotiate.
- Forcing `KexAlgorithms` in your own client confirms server acceptance and creates a controlled legacy session; it does not reproduce an unauthenticated network downgrade.
- A captured session is only a candidate for a group attack when it actually used the weak method. Assess the exact group, implementation, and attack cost before claiming recoverable session keys.
- A successful active downgrade additionally requires failed host-key verification or a separate protocol or implementation vulnerability.

## References

- [RFC 4253: key exchange](https://datatracker.ietf.org/doc/html/rfc4253)
- [RFC 9142: SSH key exchange recommendations](https://datatracker.ietf.org/doc/html/rfc9142)
- [Logjam and weak DH groups](https://weakdh.org/)
