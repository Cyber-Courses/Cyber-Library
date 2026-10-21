---
title: "Trust abuse: exploiting r-command host trust"
order: 2
description: "rsh and rlogin can authenticate by host-based trust: .rhosts (per user) and /etc/hosts.equiv (system-wide) grant passwordless access to matching host/user identities. Attacks include planting a .rhosts entry, abusing permissive or wildcard rules, exploiting hosts.equiv breadth, and spoofing a trusted source IP."
keywords:
  - trust abuse
  - rhosts
  - hosts.equiv
  - wildcard
  - ip spoofing
---

# Trust abuse

The defining weakness of `rsh` and `rlogin` is their host-based trust: `~/.rhosts` (per user) and `/etc/hosts.equiv` (system-wide) can grant passwordless access based on the connecting host and remote username. This is authentication by spoofable network identity, and it is abused four ways: planting a `.rhosts` entry where a home is writable, exploiting a permissive or wildcard rule, leveraging a broad `hosts.equiv` rule, and spoofing a trusted source IP to satisfy the check without controlling the trusted host. A host-only `hosts.equiv` entry normally permits access only to a like-named local account; an explicit username or wildcard can broaden that access. `rexec` is separate: it sends a username and password and does not use these trust files.

```bash
# read existing trust where files are accessible (reveals who is trusted)
cat ~*/.rhosts /etc/hosts.equiv 2>/dev/null
# a matching rule can yield passwordless rsh/rlogin access
```

## Subtopics

- **[rhosts write](rhosts-write.md)**: planting a trusting .rhosts entry.
- **[Wildcard trust](wildcard-trust.md)**: abusing `+ +` and over-broad entries.
- **[hosts.equiv trust](hosts-equiv-trust.md)**: exploiting system-wide trust.
- **[IP spoofing](ip-spoofing.md)**: impersonating a trusted source address.

## References

- [Linux-PAM pam_rhosts](https://man7.org/linux/man-pages/man8/pam_rhosts.8.html)
- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
