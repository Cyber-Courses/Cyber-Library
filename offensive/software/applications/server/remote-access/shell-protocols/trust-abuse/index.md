---
title: "Trust abuse: exploiting r-command host trust"
order: 2
description: "The r-commands authenticate by host-based trust: .rhosts (per user) and /etc/hosts.equiv (system-wide) grant passwordless access to listed host/user pairs, checked by source address. Attacks are planting a .rhosts entry, abusing a permissive or wildcard trust, exploiting hosts.equiv breadth, and spoofing a trusted source IP, all giving passwordless access across rsh, rlogin, and rexec."
keywords:
  - trust abuse
  - rhosts
  - hosts.equiv
  - wildcard
  - ip spoofing
---

# Trust abuse

The defining weakness of the r-commands is their host-based trust: `~/.rhosts` (per user) and `/etc/hosts.equiv` (system-wide) list `host user` pairs that are granted passwordless access, and the check is made against the connecting source address. This is authentication by spoofable network identity, and it is abused four ways that apply uniformly across rsh, rlogin, and rexec: planting a `.rhosts` entry where a home is writable, exploiting a permissive or wildcard trust already present, leveraging the breadth of `hosts.equiv` (all non-root users at once), and spoofing a trusted source IP to satisfy the check without controlling the trusted host.

```bash
# read existing trust where files are accessible (reveals who is trusted)
cat ~*/.rhosts /etc/hosts.equiv 2>/dev/null
# any of the techniques below yields passwordless rsh/rlogin/rexec
```

## Subtopics

- **[rhosts write](rhosts-write.md)**: planting a trusting .rhosts entry.
- **[Wildcard trust](wildcard-trust.md)**: abusing `+ +` and over-broad entries.
- **[hosts.equiv trust](hosts-equiv-trust.md)**: exploiting system-wide trust.
- **[IP spoofing](ip-spoofing.md)**: impersonating a trusted source address.

## References

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
