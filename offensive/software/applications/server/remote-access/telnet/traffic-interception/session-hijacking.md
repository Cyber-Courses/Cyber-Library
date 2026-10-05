---
title: "Session hijacking: taking over a live Telnet session"
description: "A Telnet session is an unencrypted, unauthenticated-after-login TCP stream, so an on-path attacker injects commands into it that run as the already-authenticated user, or takes the session over entirely. This bypasses authentication completely by riding an established session, executing with the victim's privileges on the target."
keywords:
  - session hijacking
  - tcp injection
  - telnet
  - command injection
  - mitm
---

# Session hijacking

Telnet authenticates only at login; thereafter the session is just an unencrypted TCP stream with no per-packet authentication. An on-path attacker therefore hijacks it: by injecting crafted TCP segments with the correct sequence numbers (observable because the traffic is cleartext), they insert commands into the stream that the server executes as the already-authenticated user, or desynchronise and take the session over entirely. This bypasses the login completely, the attacker never needs the password, they ride a session someone else authenticated, and runs with that user's privileges.

```bash
# on-path position to the Telnet session (ARP/route)
# observe the stream to learn the TCP sequence/ack state (cleartext makes this easy)
# inject a command segment with the right seq/ack so the server executes it as the user
# classic tooling automated TCP session hijacking against telnet/rlogin:
#   (e.g. the historic Hunt/Juggernaut tools; modern equivalents craft segments directly)
```

## Exploitation notes

- Hijacking rides an authenticated session, so it needs no credential, only an on-path position and the cleartext stream to read sequence/ack state for injection.
- Injecting a single command (add a user, write a key, start a reverse shell) is often enough and less disruptive than a full takeover, which can desynchronise and alert the user.
- The injected command runs with the victim's privileges, frequently administrative on the device, so one injected command can be a full compromise.
- This is the active end of Telnet interception; the passive counterparts are [password sniffing](password-sniffing.md) and [command interception](command-interception.md), and the same cleartext weakness enables all three.

## References

- [HackTricks: Telnet session hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [TCP session hijacking (classic technique)](https://www.rfc-editor.org/rfc/rfc793)
