---
title: "Traffic interception: capturing and hijacking cleartext Telnet"
order: 4
description: "A Telnet session that has not negotiated encryption exposes its login, commands, and output to a positioned attacker. An active on-path attacker may also attempt to inject correctly sequenced data into the live TCP connection."
keywords:
  - telnet interception
  - cleartext
  - password sniffing
  - session hijacking
  - mitm
---

# Traffic interception

When Telnet does not negotiate its encryption extension or run through another protective transport, an attacker with a network position can read the username, password, commands, and output. An active on-path attacker who can observe and inject correctly sequenced TCP data may also attempt to hijack the established session. Confirm that the data stream is plaintext before treating passive capture as credential or content disclosure.

```bash
# capture Telnet from an on-path position
tcpdump -i eth0 -w telnet.pcap port 23
# follow the TCP stream to read credentials, commands, and output
```

## Subtopics

- **[Password sniffing](password-sniffing.md)**: capturing credentials from the cleartext login.
- **[Command interception](command-interception.md)**: reading commands and output in transit.
- **[Session hijacking](session-hijacking.md)**: taking over the live Telnet session.

## References

- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
- [RFC 2946 (Telnet Data Encryption Option)](https://datatracker.ietf.org/doc/html/rfc2946)
- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
