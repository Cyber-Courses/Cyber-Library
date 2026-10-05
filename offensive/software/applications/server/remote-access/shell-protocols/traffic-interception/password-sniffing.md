---
title: "Password sniffing: capturing r-command credentials"
description: "rexec sends a username and password in cleartext with each request, and rlogin sends a cleartext password when host trust does not apply. A positioned attacker captures these directly from the traffic, obtaining reusable credentials with no guessing and nothing logged as a failed attempt."
keywords:
  - password sniffing
  - rexec
  - rlogin
  - cleartext credentials
  - capture
---

# Password sniffing

Where the r-commands use a password, rexec always, rlogin when host trust does not grant access, that password is sent in cleartext, so a positioned attacker captures it straight from the stream. rexec includes the username and password in each execution request (so every invocation is a capture opportunity), and rlogin's fallback password prompt is sent character by character and reassembled. The result is a reusable account credential obtained quietly, no brute force, no failed-login records, no lockout, and such credentials are typically reused across the legacy environment.

```bash
# capture credentials from rexec (512) and rlogin (513)
tcpdump -i eth0 -A 'port 512 or port 513' -w creds.pcap
tshark -r creds.pcap -q -z follow,tcp,ascii,0     # username/password in the stream
```

## Exploitation notes

- rexec is the richest source because it transmits the credential with every command, so scripted/automated rexec usage leaks it repeatedly; rlogin leaks only when it falls back to a password (not when host trust applies).
- Reassemble the client-to-server bytes (rlogin sends per-character) to recover the typed password; `tshark`/Wireshark do this automatically.
- Captured credentials are reusable on the host and across legacy systems; test broadly, and prefer capture over [brute force](../rlogin/cleartext-passwords.md)-style guessing wherever a position exists.
- rsh host-trust sessions usually carry no password (nothing to sniff there); target rexec/rlogin for credentials and rsh for [command/content](command-interception.md).

## References

- [HackTricks: r-commands sniffing](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
