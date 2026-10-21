---
title: "rlogin: attacking the remote login service"
order: 4
description: "rlogin (rlogind on TCP 513) opens an unencrypted interactive login session. Traditional rlogind checks for a privileged client source port and .rhosts/hosts.equiv trust, then may invoke the system login flow when automatic authentication fails. Attacks include trust abuse, traffic capture, and TCP session hijacking."
keywords:
  - rlogin
  - rlogind
  - port 513
  - rhosts
  - cleartext
---

# rlogin

`rlogin` opens an interactive login session on a remote host via `rlogind` on TCP 513. Traditional implementations require the client connection to originate from a privileged port in the 512–1023 range, then evaluate host-based trust through `~/.rhosts` and `/etc/hosts.equiv`. If automatic authentication fails, `rlogind` may invoke the configured login program over the same unencrypted terminal stream; whether that flow requests a password, uses another PAM mechanism, or rejects access depends on the target. The attacks follow: passwordless login by abusing or planting trust, capture of credentials and session data when an unencrypted login flow is used, implementation-dependent username enumeration, and hijacking the live TCP session. Because it grants an interactive shell, a successful rlogin is a direct foothold.

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
