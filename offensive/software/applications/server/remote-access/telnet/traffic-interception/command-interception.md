---
title: "Command interception: reading Telnet commands and output in transit"
description: "When Telnet encryption is not active, commands and server output cross the network in plaintext. A positioned attacker can reconstruct that stream and recover configuration, command results, and secrets entered during the session."
keywords:
  - command interception
  - session capture
  - cleartext
  - output disclosure
  - telnet
---

# Command interception

When the Telnet stream remains plaintext, command interception exposes the interaction after login: commands, server output, secondary credentials typed during the session, and data viewed by the operator. A positioned attacker can reconstruct both directions of the TCP stream. If Telnet encryption or another protective transport is active, the capture does not reveal this application data directly.

```bash
# capture and reconstruct the full session
tcpdump -i eth0 -w telnet.pcap port 23
# Wireshark Follow TCP Stream shows the entire session (commands + output)
tshark -r telnet.pcap -q -z follow,tcp,ascii,0
```

## Exploitation notes

- Session capture frequently surpasses the login in value: `enable`/`su`/`sudo` passwords entered mid-session, credentials for other systems, device configuration (including secrets in a `show running-config`), and sensitive output all appear in the clear.
- Reconstruct the full bidirectional stream to pair commands with their output; the client-to-server direction holds what the user typed, the server-to-client direction holds results.
- This is passive and quiet; combine with [password sniffing](password-sniffing.md) from the same capture for the login plus everything after.
- For active control rather than observation, escalate to [session hijacking](session-hijacking.md).

## References

- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
