---
title: "Session hijacking: taking over a live rlogin session"
order: 2
description: "An rlogin session is an unencrypted TCP stream authenticated only at setup, so an on-path attacker injects commands into it or takes it over, executing as the logged-in user. This is the classic r-command/Telnet TCP session-hijacking attack, bypassing authentication by riding an established session."
keywords:
  - session hijacking
  - rlogin
  - tcp injection
  - hunt
  - juggernaut
---

# Session hijacking

Once an rlogin session is established, it is just an unencrypted TCP stream with no per-packet authentication, so an on-path attacker hijacks it. By reading the cleartext stream to learn the TCP sequence and acknowledgement state, the attacker injects segments carrying commands that the server executes as the logged-in user, or desynchronises the legitimate client and takes the session over entirely. This bypasses authentication completely, the attacker never needs trust or a password, they commandeer a session another user authenticated, running with that user's privileges. rlogin was a canonical target of the classic session-hijacking tools.

```bash
# on-path position to the rlogin session (ARP/route); the cleartext stream reveals
# the TCP seq/ack state. Inject a command segment the server runs as the user:
#   e.g. append trust for durable re-entry
#   echo "+ +" >> ~/.rhosts
# classic automated tools (Hunt, Juggernaut) implemented rlogin/telnet TCP hijacking;
# modern exploitation crafts the segments directly from the observed seq/ack.
```

## Exploitation notes

- Injecting a single command (plant `.rhosts`, add a user, start a reverse shell) is usually the goal; a full takeover desynchronises and may alert the live user, whereas a quiet injection establishes durable access.
- The injected command runs as the logged-in user, often administrative on legacy Unix, so one command can be a full compromise.
- It needs only an on-path position and the cleartext stream, no credential or trust; it is the active counterpart to passively [capturing the session](cleartext-passwords.md).
- The identical technique applies to rsh and Telnet; the shared cleartext, setup-only-auth design is the root cause.

## References

- [HackTricks: rlogin hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
