---
title: "PPTP: the broken MS-CHAPv2 authentication"
order: 3
description: "PPTP (TCP 1723) authenticates with MS-CHAPv2, whose security reduces to a single DES key that is brute-forced in bounded time, so a captured PPTP handshake is cracked to recover the password or the MPPE key. PPTP is considered cryptographically broken, and a captured authentication reliably yields the credential offline."
keywords:
  - pptp
  - ms-chapv2
  - chapcrack
  - des
  - mppe
---

# PPTP

PPTP (Point-to-Point Tunneling Protocol, TCP 1723 with GRE for data) authenticates with MS-CHAPv2, which is cryptographically broken. The MS-CHAPv2 response is derived from the NT hash in a way that reduces, through the protocol's use of three DES operations with a known structure, to recovering a single DES key, which is brute-forceable in bounded time regardless of password strength (the password strength does not matter because the attack targets the hash-derived key). A captured PPTP handshake is therefore cracked offline to recover the password/NT hash and the MPPE encryption key, decrypting the tunnel. PPTP is deprecated precisely because of this.

```bash
# capture a PPTP MS-CHAPv2 handshake (on-path or by prompting a connection)
tcpdump -i eth0 -w pptp.pcap tcp port 1723 or proto gre
# extract the MS-CHAPv2 challenge/response and crack to the DES key / NT hash
chapcrack parse -i pptp.pcap                     # extracts the handshake fields
# the reduced DES keyspace is brute-forced in bounded time (historically via cloud crackers)
```

## Exploitation notes

- The attack is independent of password strength: MS-CHAPv2's structure lets the response be reduced to one DES key brute-forceable in bounded time, so any captured handshake is crackable to the NT hash and MPPE key.
- A captured handshake yields both the credential (NT hash, usable for pass-the-hash elsewhere) and the key to decrypt the PPTP tunnel traffic.
- Capture needs an on-path position or the ability to induce a connection; PPTP's control channel (1723) and GRE data are both sniffable.
- Treat any PPTP endpoint as effectively unauthenticated-grade crypto; it is deprecated and should be assumed breakable once a handshake is observed.

## Tools

- [chapcrack](https://github.com/moxie0/chapcrack)

## References

- [MS-CHAPv2 cryptanalysis (Marlinspike/Hulton)](https://www.cloudcracker.com/blog/2012/07/29/cracking-ms-chap-v2/)
- [HackTricks: PPTP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-pptp)
