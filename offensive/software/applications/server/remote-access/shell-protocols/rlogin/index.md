---
title: "rlogin: attacking the remote login service"
description: "rlogin (rlogind on TCP 513) opens an interactive login session, authenticating by the same .rhosts/hosts.equiv host trust as rsh, or falling back to a cleartext password. Attacks are passwordless login through trust abuse, capture of the cleartext password and session, username enumeration from the login behaviour, and session hijacking."
keywords:
  - rlogin
  - rlogind
  - port 513
  - rhosts
  - cleartext
---

# rlogin

`rlogin` opens an interactive login session on a remote host via `rlogind` on TCP 513. It authenticates the same way as rsh, host-based trust through `~/.rhosts` and `/etc/hosts.equiv`, and falls back to a cleartext password prompt when trust does not apply. The attacks follow: passwordless login by abusing or planting trust, capture of the cleartext password and the entire session when a password is used, username enumeration from how the login responds, and hijacking the live cleartext session. Because it grants an interactive shell, a successful rlogin is a direct foothold.

```bash
# trusted login (no password) or password login
rlogin -l <user> <target>
nmap -p513 -sV <target>
```

## Subtopics

- **[rhosts bypass](rhosts-bypass.md)**: passwordless login via host trust.
- **[Cleartext passwords](cleartext-passwords.md)**: capturing the password when trust is not used.
- **[Username enumeration](username-enumeration.md)**: discovering valid users from login behaviour.
- **[Session hijacking](session-hijacking.md)**: taking over the live rlogin session.

## References

- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
- [HackTricks: rlogin (513)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
