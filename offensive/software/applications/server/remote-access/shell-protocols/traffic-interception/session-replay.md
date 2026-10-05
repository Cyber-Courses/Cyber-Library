---
title: "Session replay: replaying captured r-command sessions"
description: "Because the r-commands authenticate only at setup and carry sessions in cleartext, an attacker who captures the traffic can hijack the live TCP session to inject commands as the authenticated user, and in some cases replay captured exchanges. This executes commands without defeating authentication, riding a session someone else established."
keywords:
  - session replay
  - tcp hijacking
  - command injection
  - rsh
  - rlogin
---

# Session replay

The r-commands authenticate only during connection setup, after which the session is an unauthenticated cleartext TCP stream. An attacker who captures or sits on that stream exploits this two ways. The direct and reliable one is live session hijacking: reading the cleartext stream to learn the TCP sequence/ack state and injecting segments carrying commands the server executes as the authenticated user. The second is replay of captured exchanges where the server does not prevent it. Both execute commands without defeating authentication, by riding a session another party established, and a single injected command can plant durable trust.

```bash
# live hijack: on-path, read seq/ack from the cleartext stream, inject a command
#   e.g. append trust so later access needs no interception
#   echo "+ +" >> ~/.rhosts
# classic tools (Hunt, Juggernaut) automated rsh/rlogin TCP hijacking; modern
# exploitation crafts the injection segments directly from the observed stream.
```

## Exploitation notes

- Live hijacking is the practical form: inject one command (plant `.rhosts`, add a user, start a reverse shell) into the established session, running as the authenticated user, often administrative on legacy Unix.
- A quiet single-command injection is preferred over a full takeover, which desynchronises and may alert the user.
- It needs only an on-path position and the cleartext stream, no credential or trust; it is the active counterpart to [command interception](command-interception.md) and the same technique as [rlogin session hijacking](../rlogin/session-hijacking.md) and Telnet.
- The shared cleartext, setup-only-authentication design across rsh/rlogin/rexec is the root cause enabling all interception and replay.

## References

- [HackTricks: r-commands hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
