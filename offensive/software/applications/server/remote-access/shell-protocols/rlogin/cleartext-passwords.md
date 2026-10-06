---
title: "Cleartext passwords: capturing the rlogin login"
order: 1
description: "When host trust does not apply, rlogin falls back to a password prompt sent in cleartext. A positioned attacker captures the username and password directly from the traffic, and the entire interactive session along with it, yielding a reusable credential with no guessing and nothing logged as a failed attempt."
keywords:
  - cleartext password
  - rlogin
  - sniffing
  - credential capture
  - session
---

# Cleartext passwords

If the host-trust check does not grant access, `rlogin` prompts for a password, and like everything in the r-commands that password crosses the network unencrypted. An attacker on the path captures the username and password straight from the stream, plus the full interactive session that follows. This gives a reusable credential with none of the noise of brute force, no failed-login records, no lockout, and the captured account is typically reused across the legacy environment where r-commands persist.

```bash
# capture the cleartext rlogin login and session
tcpdump -i eth0 -A port 513 -w rlogin.pcap
tshark -r rlogin.pcap -q -z follow,tcp,ascii,0     # reassemble login + session
```

## Exploitation notes

- Capture beats guessing: a sniffed rlogin login yields the exact credential quietly, so prefer it wherever an on-path position exists, the same reasoning as Telnet password sniffing.
- The whole session is captured too, exposing commands, output, and any secondary credentials entered, see [command/session capture](../traffic-interception/index.md).
- The credential is typically reusable on other legacy hosts and sometimes other services; test it broadly.
- Needs an on-path/tap position; combine with trust abuse (which needs no password at all) depending on whether trust or a password is in use.

## References

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
