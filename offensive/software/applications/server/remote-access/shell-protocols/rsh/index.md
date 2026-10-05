---
title: "rsh: attacking the remote shell service"
description: "rsh (rshd on TCP 514) executes a command or opens a shell on a remote host, authenticating by the host-based trust in .rhosts and /etc/hosts.equiv rather than a password. An attacker abuses that trust, by writing a .rhosts file, exploiting a permissive hosts.equiv, or spoofing a trusted source IP, to run commands with no credential, and intercepts the cleartext session."
keywords:
  - rsh
  - rshd
  - port 514
  - rhosts
  - remote shell
---

# rsh

`rsh` runs a command (or an interactive shell) on a remote host via `rshd` on TCP 514. Its authentication is host-based: the server checks whether the connecting source IP and username are trusted via the target user's `~/.rhosts` or the system-wide `/etc/hosts.equiv`, and if so, grants access with no password. This design is the whole vulnerability. An attacker who can make the server trust them, by writing a `.rhosts` entry, abusing a permissive `hosts.equiv`, or spoofing a trusted source address, runs commands as the target user with no credential, and because the channel is cleartext, any real session is also interceptable.

```bash
# if trusted (or trust is permissive), run commands with no password
rsh -l <user> <target> id
rsh -l root <target> 'cat /etc/shadow'
# test reachability
nmap -p514 -sV <target>
```

## Subtopics

- **[rhosts bypass](rhosts-bypass.md)**: passwordless access via .rhosts trust.
- **[hosts.equiv](hosts-equiv.md)**: system-wide trust abuse.
- **[IP spoofing](ip-spoofing.md)**: impersonating a trusted source address.
- **[Cleartext interception](cleartext-interception.md)**: capturing the unencrypted session.

## References

- [rshd manual](https://linux.die.net/man/8/rshd)
- [HackTricks: rsh (514)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
