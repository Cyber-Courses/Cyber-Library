---
title: "Command interception: reading r-command content in transit"
description: "The r-commands carry the executed command, its arguments, and all output in cleartext. A positioned attacker reconstructs these from the traffic, capturing what was run, the results, and any sensitive data or secondary credentials handled during the session, often yielding more than the login itself."
keywords:
  - command interception
  - cleartext
  - output capture
  - rsh
  - sniffing
---

# Command interception

Every r-command exchange carries its payload in cleartext: for rsh/rexec the command and arguments and their output, and for rlogin the whole interactive session. A positioned attacker reconstructs all of it, learning exactly what was executed, seeing the results, and capturing any sensitive data or secondary credentials that pass through (a command that prints a config with secrets, a `su`/`sudo` password typed in an rlogin session, data read from files). This frequently yields more than a captured login, because administrative commands and their output expose the systems and secrets the operator was working with.

```bash
# capture and reconstruct the command/output content
tcpdump -i eth0 -A 'port 513 or port 514' -w content.pcap
tshark -r content.pcap -q -z follow,tcp,ascii,0     # command (c->s) and output (s->c)
```

## Exploitation notes

- Content capture often beats the login: executed commands and their output reveal configuration, secrets printed to the terminal, and secondary credentials entered mid-session.
- For rsh/rexec the key direction is client-to-server (the command) and server-to-client (the output); for rlogin reconstruct the full bidirectional session.
- This is passive and quiet; pair with [password sniffing](password-sniffing.md) from the same capture for credentials plus content, and with [session replay](session-replay.md) or hijacking for active use.
- Needs an on-path/tap position; legacy r-command segments are typically flat and easily intercepted.

## References

- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
