---
title: "Password sniffing: capturing Telnet credentials from the wire"
description: "When Telnet encryption is not negotiated, a positioned attacker can reconstruct the username and password from the TCP stream. Verify that the session is plaintext before assuming its credentials are exposed."
keywords:
  - password sniffing
  - cleartext credentials
  - tcpdump
  - wireshark
  - telnet
---

# Password sniffing

When the Telnet data stream remains plaintext, capturing a real authentication can reveal the credential without password guessing or failed-login noise. Interactive input often appears as small segments or individual characters, so the client-to-server stream must be reconstructed. If the ENCRYPT option or another protective transport is active, this technique does not expose the plaintext password.

```bash
# capture and reassemble a Telnet login from an on-path position
tcpdump -i eth0 -A port 23
# Wireshark: Follow TCP Stream reconstructs the full login (and session)
# reconstruct the first TCP stream from a pcap; select the relevant stream number
tshark -r telnet.pcap -q -z follow,tcp,ascii,0
```

## Exploitation notes

- Reassembly matters: because input is per-character (and echoed), read the client-to-server bytes to recover what was typed; Wireshark's Follow TCP Stream and `tshark` do this directly.
- This captures the real credential with nothing logged as a failed attempt, far quieter than [brute force](../authentication/password-brute-force.md), so prefer it whenever a capture position exists.
- The captured login is typically a device-administrative account and commonly reused on other systems and services; test it broadly.
- Needs an on-path or tap position (ARP/DNS/route on a flat segment, a SPAN port, or a compromised intermediary).

## References

- [HackTricks: Telnet sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
