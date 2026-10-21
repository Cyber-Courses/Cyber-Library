---
title: "Traffic interception: capturing and hijacking cleartext r-commands"
order: 1
description: "The r-commands encrypt nothing, so a positioned attacker can capture rexec and rlogin passwords, commands, output, and interactive sessions. A live cleartext stream may also be hijacked through valid TCP injection; an old capture cannot simply be replayed as a new authenticated session."
keywords:
  - traffic interception
  - cleartext
  - password sniffing
  - command capture
  - session hijacking
---

# Traffic interception

None of the r-commands encrypt their traffic, so an on-path attacker can read what they carry: the cleartext passwords sent by `rexec` and password-fallback `rlogin`, commands and output, and complete interactive sessions. Beyond passive capture, an attacker who remains on path may inject correctly sequenced data into a live TCP stream after authentication. A saved TCP exchange cannot simply be replayed as a new authenticated session because a new connection has different TCP state and must satisfy the service's authentication again.

```bash
# capture all TCP traffic for the target so negotiated secondary streams are included
tcpdump -i eth0 -w rcmds.pcap 'tcp and host <target>'
# enumerate streams before following the relevant service and secondary connections
tshark -r rcmds.pcap -q -z conv,tcp
tshark -r rcmds.pcap -q -z follow,tcp,ascii,<stream-number>
```

## Subtopics

- **[Password sniffing](password-sniffing.md)**: capturing credentials from the cleartext exchange.
- **[Command interception](command-interception.md)**: reading commands and output in transit.
- **[Live session hijacking](session-replay.md)**: injecting correctly sequenced data into an established connection.

## References

- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin, cleartext)](https://datatracker.ietf.org/doc/html/rfc1282)
