---
title: "Shell protocols: attacking the Berkeley r-commands"
order: 4
description: "The Berkeley r-commands, rsh (514), rlogin (513), and rexec (512), provide legacy remote shell, login, and command execution without encryption. rsh and rlogin can use spoofable host trust, while rexec sends a username and password in cleartext."
keywords:
  - r-commands
  - rsh
  - rlogin
  - rexec
  - rhosts
---

# Shell protocols

The Berkeley r-commands are a family of legacy Unix remote-access services: `rsh` (remote shell, TCP 514), `rlogin` (remote login, TCP 513), and `rexec` (remote execution, TCP 512). They predate SSH and provide no transport encryption, so credentials, commands, output, and interactive sessions exposed by each protocol travel in cleartext. Their authentication models differ: `rsh` and `rlogin` can use `~/.rhosts` and `/etc/hosts.equiv` to grant passwordless access based on the client's host and username, whereas `rexec` sends a username and password with the request. The attack surface therefore includes host-trust abuse against `rsh` and `rlogin`, traffic interception across all three services, and service-specific execution paths.

```bash
nmap -p512,513,514 -sV <target>                # rexec, rlogin, rsh
# the services are often found together on legacy Unix
```

## Subtopics

- **[rsh](rsh/index.md)**: remote shell on 514 and its trust abuse.
- **[rlogin](rlogin/index.md)**: remote login on 513.
- **[rexec](rexec/index.md)**: remote execution on 512.
- **[Trust abuse](trust-abuse/index.md)**: `.rhosts` and `hosts.equiv` exploitation against rsh and rlogin.
- **[Traffic interception](traffic-interception/index.md)**: cleartext capture and live-session hijacking.

## References

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rexec/rlogin/rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
- [Unix r-commands and .rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
