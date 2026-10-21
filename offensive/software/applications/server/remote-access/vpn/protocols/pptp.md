---
title: "PPTP: the broken MS-CHAPv2 authentication"
order: 3
description: "PPTP (TCP 1723) commonly authenticates with MS-CHAPv2, whose security can be reduced to a single DES key search. A captured handshake can yield the NT password hash material used to derive the MPPE keys, enabling tunnel decryption without inherently revealing the plaintext password."
keywords:
  - pptp
  - ms-chapv2
  - chapcrack
  - des
  - mppe
---

# PPTP

PPTP (Point-to-Point Tunneling Protocol, TCP 1723 with GRE for data) commonly authenticates with MS-CHAPv2, which is cryptographically broken. The MS-CHAPv2 response is derived from the NT password hash in a way that reduces, through the protocol's use of three DES operations with a known structure, to recovering a single DES key. That search is independent of the original password's strength because it targets the hash-derived protocol value. A captured handshake can therefore yield the NT hash material needed to derive the MPPE encryption keys and decrypt captured tunnel traffic. It does not inherently reveal the plaintext password; recovering that still requires a separate password-cracking step.

```bash
# capture a PPTP MS-CHAPv2 handshake (on-path or by prompting a connection)
tcpdump -i eth0 -w pptp.pcap tcp port 1723 or proto gre
# extract the MS-CHAPv2 challenge/response and crack to the DES key / NT hash
chapcrack parse -i pptp.pcap                     # extracts the handshake fields
# the reduced DES keyspace is brute-forced in bounded time (historically via cloud crackers)
```

## Exploitation notes

- The protocol attack is independent of password strength: MS-CHAPv2's structure reduces the relevant search to one DES key and can recover the NT hash material used to derive the MPPE keys.
- The recovered NT hash may be usable for authentication against other systems that accept NTLM-derived credentials, but it is not the plaintext password; obtaining the password requires a separate cracking step.
- Capture needs an on-path position or the ability to induce a connection; PPTP's control channel (1723) and GRE data are both sniffable.
- Treat any PPTP endpoint as effectively unauthenticated-grade crypto; it is deprecated and should be assumed breakable once a handshake is observed.

## Tools

- [chapcrack](https://github.com/moxie0/chapcrack)

## References

- [MS-CHAPv2 cryptanalysis (Marlinspike/Hulton)](https://www.cloudcracker.com/blog/2012/07/29/cracking-ms-chap-v2/)
- [HackTricks: PPTP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-pptp)
