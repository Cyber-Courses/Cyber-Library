---
title: "Shell protocols: attacking the Berkeley r-commands"
order: 4
description: "The Berkeley r-commands, rsh (514), rlogin (513), and rexec (512), provide remote shell, login, and command execution with no encryption and a host-based trust model. Their weaknesses are the .rhosts and hosts.equiv trust that authenticates by source address (spoofable), cleartext credentials and sessions, and command execution through trusted-host relationships."
keywords:
  - r-commands
  - rsh
  - rlogin
  - rexec
  - rhosts
---

# Shell protocols

The Berkeley r-commands are a family of legacy Unix remote-access services: `rsh` (remote shell, TCP 514), `rlogin` (remote login, TCP 513), and `rexec` (remote execution, TCP 512). They predate SSH and share two fatal weaknesses. First, no encryption: credentials, commands, and sessions travel in cleartext. Second, a host-based trust model, `.rhosts` and `/etc/hosts.equiv` files that grant passwordless access based on the client's source IP and username, which is authentication by spoofable network identity. The attack surface is therefore trust abuse (forging or planting trust to get passwordless shells), cleartext interception, and the per-command execution paths.

```bash
nmap -p512,513,514 -sV <target>                # rexec, rlogin, rsh
# the services are often found together on legacy Unix
```

## Subtopics

- **[rsh](rsh/index.md)**: remote shell on 514 and its trust abuse.
- **[rlogin](rlogin/index.md)**: remote login on 513.
- **[rexec](rexec/index.md)**: remote execution on 512.
- **[Trust abuse](trust-abuse/index.md)**: .rhosts and hosts.equiv exploitation across the r-commands.
- **[Traffic interception](traffic-interception/index.md)**: cleartext capture and replay.

## References

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rexec/rlogin/rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
- [Unix r-commands and .rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
