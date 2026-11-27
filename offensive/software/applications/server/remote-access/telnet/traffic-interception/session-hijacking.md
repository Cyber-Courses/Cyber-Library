---
title: "Session hijacking: taking over a live Telnet session"
description: "When Telnet lacks an encryption or integrity layer, an active on-path attacker who can track live TCP state may attempt to inject data into an established session. Success and interpretation depend on TCP synchronization and the active application context."
keywords:
  - session hijacking
  - tcp injection
  - telnet
  - command injection
  - mitm
---

# Session hijacking

After login, Telnet application data is carried over a TCP stream without application-level integrity protection. An active on-path attacker who can observe the current sequence state and inject packets may insert data that the server processes in the authenticated session or may desynchronise the legitimate client. Plaintext makes observing the interaction easier, but the attack additionally requires live packet-injection capability; a saved capture alone is insufficient.

```bash
# on-path position to the Telnet session (ARP/route)
# observe the stream to learn the TCP sequence/ack state (cleartext makes this easy)
# inject a command segment with the right seq/ack so the server executes it as the user
# classic tooling automated TCP session hijacking against telnet/rlogin:
#   (e.g. the historic Hunt/Juggernaut tools; modern equivalents craft segments directly)
```

## Exploitation notes

- Hijacking rides an authenticated session, so it does not require the user's credential, but it does require an active on-path position, current TCP state, viable packet injection, and usually a way to handle or suppress competing traffic.
- Injecting a single command (add a user, write a key, start a reverse shell) is often enough and less disruptive than a full takeover, which can desynchronise and alert the user.
- If injected bytes reach an interactive command context, resulting actions run with that session's privileges. A restricted menu, non-shell context, insufficient privileges, or failed synchronization can substantially reduce the impact.
- This is the active end of Telnet interception; the passive counterparts are [password sniffing](password-sniffing.md) and [command interception](command-interception.md), and the same cleartext weakness enables all three.

## References

- [HackTricks: Telnet session hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [TCP session hijacking (classic technique)](https://www.rfc-editor.org/rfc/rfc793)
