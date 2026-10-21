---
title: "Password sniffing: capturing r-command credentials"
description: "rexec sends a username and password in cleartext with each request. When failed rlogin trust leads the target's login flow to request a password, that credential also crosses the unencrypted terminal stream. A positioned attacker can capture credentials used in either case."
keywords:
  - password sniffing
  - rexec
  - rlogin
  - cleartext credentials
  - capture
---

# Password sniffing

`rexec` includes its username and password in each execution request, so every invocation is a capture opportunity. When automatic rlogin trust fails, `rlogind` may invoke a configured login or PAM flow over the same unencrypted terminal stream; if that flow requests a password, a positioned attacker can reassemble the client-to-server characters and capture it. Other targets may use a different authentication mechanism or reject access without requesting a password. Passive capture avoids failed guesses and lockouts, but reuse on other hosts or services exists only where account provisioning or password policy makes it possible.

```bash
# capture credentials from rexec (512) and rlogin (513)
tcpdump -i eth0 -w creds.pcap 'port 512 or port 513'
tshark -r creds.pcap -q -z follow,tcp,ascii,0     # username/password in the stream
```

## Exploitation notes

- rexec transmits the credential with every command, so scripted or automated usage leaks it repeatedly; rlogin exposes a password only when the target's post-trust login flow actually requests one.
- Reassemble the client-to-server bytes (rlogin sends per-character) to recover the typed password; `tshark`/Wireshark do this automatically.
- A captured password may work elsewhere only when credentials are shared or synchronized; do not assume cross-host reuse, and assess it only within the authorized scope.
- rsh host-trust sessions usually carry no password (nothing to sniff there); target rexec/rlogin for credentials and rsh for [command/content](command-interception.md).

## References

- [HackTricks: r-commands sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
