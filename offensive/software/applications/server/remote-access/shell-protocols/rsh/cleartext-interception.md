---
title: "Cleartext interception: capturing the unencrypted rsh session"
order: 1
description: "rsh encrypts nothing, so a positioned attacker can capture the executed command, its output, and any transferred data directly from the wire. Traditional rsh authenticates through host trust rather than transmitting a password, and its live TCP stream may be susceptible to on-path injection."
keywords:
  - cleartext
  - rsh
  - command capture
  - sniffing
  - interception
---

# Cleartext interception

Like the other r-commands, `rsh` provides no encryption, so an attacker on the path can read the command, its output, and transferred data. Traditional `rshd` authenticates the requested user through `ruserok` and the `hosts.equiv`/`.rhosts` trust model; it does not provide an `rlogin`-style password fallback. The capture may still expose sensitive data or secondary credentials handled by the command, and an attacker able to inject correctly sequenced packets into the live TCP stream may attempt a session takeover.

```bash
# capture all TCP traffic for the target so the negotiated stderr stream is included
tcpdump -i eth0 -w rsh.pcap 'tcp and host <target>'
# enumerate streams, then follow the rsh control/stdin/stdout stream and any stderr stream
tshark -r rsh.pcap -q -z conv,tcp
tshark -r rsh.pcap -q -z follow,tcp,ascii,<stream-number>
```

## Exploitation notes

- The main capture value is the command and its output, which frequently carry sensitive data or secondary credentials; host-trust rsh often has no password to sniff, so content is the target.
- Correctly sequenced packets may inject input into the process that `rshd` already started. That input becomes a shell command only when the active remote process is an interactive shell or another interpreter; a fixed command such as `id` will not treat arbitrary injected text as a new command. Continued injection also has to manage TCP sequence desynchronization.
- Needs an on-path or tap position; r-commands on a flat legacy segment are readily intercepted.
- This is the shared cleartext weakness of all r-commands; see the [traffic-interception](../traffic-interception/index.md) group for the cross-command treatment.

## References

- [HackTricks: rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin, same cleartext model)](https://datatracker.ietf.org/doc/html/rfc1282)
