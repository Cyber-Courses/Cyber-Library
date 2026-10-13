---
title: "Traffic interception: capturing and replaying cleartext r-commands"
order: 1
description: "The r-commands encrypt nothing, so a positioned attacker captures the passwords rexec/rlogin send, every command and its output, and whole sessions. The cleartext, setup-only-authenticated streams can be hijacked to inject commands and replayed, giving credentials, data, and execution without defeating authentication."
keywords:
  - traffic interception
  - cleartext
  - password sniffing
  - command capture
  - session replay
---

# Traffic interception

None of the r-commands encrypt their traffic, so an on-path attacker reads everything they carry: the cleartext passwords that rexec and password-fallback rlogin send, every command executed and its full output, and complete interactive sessions. Beyond passive capture, the streams are authenticated only at setup, so they can be hijacked to inject commands as the authenticated user, and captured sessions can be replayed. Interception is frequently the easiest attack on legacy r-command deployments because it needs no credential guessing or trust manipulation, only a position on the network.

```bash
# capture across the r-command ports
tcpdump -i eth0 -A 'port 512 or port 513 or port 514' -w rcmds.pcap
tshark -r rcmds.pcap -q -z follow,tcp,ascii,0
```

## Subtopics

- **[Password sniffing](password-sniffing.md)**: capturing credentials from the cleartext exchange.
- **[Command interception](command-interception.md)**: reading commands and output in transit.
- **[Session replay](session-replay.md)**: replaying captured r-command sessions.

## References

- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin, cleartext)](https://datatracker.ietf.org/doc/html/rfc1282)
