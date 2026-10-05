---
title: "Cleartext interception: capturing the unencrypted rsh session"
description: "rsh encrypts nothing, so a positioned attacker captures the executed commands, their output, and any data in the session directly from the wire. Where password-style authentication is used rather than host trust, the credential is captured too, and the session can be hijacked as with other cleartext remote-shell protocols."
keywords:
  - cleartext
  - rsh
  - command capture
  - sniffing
  - interception
---

# Cleartext interception

Like the other r-commands, `rsh` provides no encryption, so an attacker on the path reads the entire exchange: the command executed, its full output, and any data transferred. Because rsh commonly relies on host-based trust rather than passwords, the prize is usually the command and output (which often include sensitive data or secondary credentials the command handles) rather than a login password, but where password authentication is in play it is captured too. The unauthenticated-after-setup TCP stream is also hijackable, matching the Telnet/rlogin session-takeover technique.

```bash
# capture rsh traffic from an on-path position
tcpdump -i eth0 -A port 514 -w rsh.pcap
# follow the TCP stream to read the command, output, and any data/credentials
tshark -r rsh.pcap -q -z follow,tcp,ascii,0
```

## Exploitation notes

- The main capture value is the command and its output, which frequently carry sensitive data or secondary credentials; host-trust rsh often has no password to sniff, so content is the target.
- The stream is hijackable (inject a command into the established TCP session), the same technique as [rlogin session hijacking](../rlogin/session-hijacking.md) and Telnet; a single injected command can plant durable `.rhosts` trust.
- Needs an on-path or tap position; r-commands on a flat legacy segment are readily intercepted.
- This is the shared cleartext weakness of all r-commands; see the [traffic-interception](../traffic-interception/index.md) group for the cross-command treatment.

## References

- [HackTricks: rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin, same cleartext model)](https://datatracker.ietf.org/doc/html/rfc1282)
