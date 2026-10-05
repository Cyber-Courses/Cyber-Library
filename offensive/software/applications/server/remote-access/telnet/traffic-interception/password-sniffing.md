---
title: "Password sniffing: capturing Telnet credentials from the wire"
description: "The Telnet login is cleartext, so a positioned attacker captures the username and password directly from the traffic, with no guessing and nothing logged on the server. Keystrokes are sent character by character, so the credential is reassembled from the stream, and the captured account is typically reusable."
keywords:
  - password sniffing
  - cleartext credentials
  - tcpdump
  - wireshark
  - telnet
---

# Password sniffing

Telnet transmits the login in cleartext, so capturing a real authentication yields the exact credential with no brute force and no failed-login noise on the server. The one wrinkle is that Telnet sends input character by character (and the server often echoes it), so the username and password appear as a stream of single bytes that must be reassembled, which standard tools do automatically. A sniffed Telnet credential is the quietest way to obtain access, and the account is usually administrative on the device and reused elsewhere.

```bash
# capture and reassemble a Telnet login from an on-path position
tcpdump -i eth0 -A port 23
# Wireshark: Follow TCP Stream reconstructs the full login (and session)
# scripted extraction of credentials from a pcap
tshark -r telnet.pcap -Y 'telnet' -T fields -e telnet.data | tr -d '\n'
```

## Exploitation notes

- Reassembly matters: because input is per-character (and echoed), read the client-to-server bytes to recover what was typed; Wireshark's Follow TCP Stream and `tshark` do this directly.
- This captures the real credential with nothing logged as a failed attempt, far quieter than [brute force](../authentication/password-brute-force.md), so prefer it whenever a capture position exists.
- The captured login is typically a device-administrative account and commonly reused on other systems and services; test it broadly.
- Needs an on-path or tap position (ARP/DNS/route on a flat segment, a SPAN port, or a compromised intermediary).

## References

- [HackTricks: Telnet sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
