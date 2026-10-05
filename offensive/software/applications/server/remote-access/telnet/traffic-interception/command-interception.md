---
title: "Command interception: reading Telnet commands and output in transit"
description: "Beyond the login, a Telnet session's entire contents, the commands the user runs and the server's output, cross the network in cleartext. A positioned attacker reconstructs the full session from the traffic, capturing configuration, secondary credentials entered during the session, command results, and sensitive data the user views."
keywords:
  - command interception
  - session capture
  - cleartext
  - output disclosure
  - telnet
---

# Command interception

Password sniffing captures the login; command interception captures everything after it. Since the whole Telnet session is cleartext, an attacker on the path reconstructs the complete interaction: every command the user runs, the server's full output, any secondary credentials typed during the session (an `enable` password on a router, a `su`/`sudo` password, credentials for another system), and all data the user views. This often yields more than the login itself, because operators run privileged commands and enter further secrets inside the session.

```bash
# capture and reconstruct the full session
tcpdump -i eth0 -A port 23 -w telnet.pcap
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
