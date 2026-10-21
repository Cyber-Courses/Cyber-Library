---
title: "Cleartext passwords: capturing the rlogin login"
order: 1
description: "When automatic host-trust authentication fails, rlogind may invoke a login flow over its unencrypted terminal stream. If that flow requests a password, a positioned attacker can capture the credential and the interactive session directly from the traffic."
keywords:
  - cleartext password
  - rlogin
  - sniffing
  - credential capture
  - session
---

# Cleartext passwords

If automatic host-trust authentication fails, traditional `rlogind` invokes the configured login program over its unencrypted pseudo-terminal. The resulting authentication flow is implementation- and policy-dependent: it may request a password, use another PAM mechanism, or reject access. When a password is requested, an attacker on the path can capture the username and password from the stream, along with the interactive session that follows. Passive capture avoids failed guesses and lockouts, but the credential's validity and reuse remain limited by the target's account policy.

```bash
# capture the cleartext rlogin login and session
tcpdump -i eth0 -w rlogin.pcap port 513
tshark -r rlogin.pcap -q -z follow,tcp,ascii,0     # reassemble login + session
```

## Exploitation notes

- Capture beats guessing: a sniffed rlogin login yields the exact credential quietly, so prefer it wherever an on-path position exists, the same reasoning as Telnet password sniffing.
- The whole session is captured too, exposing commands, output, and any secondary credentials entered, see [command/session capture](../traffic-interception/index.md).
- The credential may be reused elsewhere only if accounts or passwords are shared; assess that possibility solely within the authorized scope rather than assuming reuse.
- Needs an on-path/tap position and a target login flow that actually requests a password; combine with trust abuse depending on the authentication path in use.

## References

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
