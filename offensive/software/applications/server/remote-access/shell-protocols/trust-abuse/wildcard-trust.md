---
title: "Wildcard trust: abusing + + and over-broad r-command entries"
order: 2
description: "A + token in .rhosts or hosts.equiv is a wildcard matching any host or any user, so an entry like + + trusts everyone for passwordless access. Over-broad entries, a bare +, a wildcard host, or a netgroup that resolves widely, let an attacker from any (or a spoofed) address log in without a password."
keywords:
  - wildcard
  - + +
  - rhosts
  - hosts.equiv
  - open trust
---

# Wildcard trust

The r-command trust files treat `+` as a wildcard: `+` in the host field matches any host, `+` in the user field matches any user, and the infamous `+ +` entry trusts every host and every user for passwordless access. Over-broad trust is a common real-world misconfiguration: a bare `+`, a wildcard host entry, or a netgroup that resolves to a wide set. Where such an entry exists in a user's `.rhosts` or in `/etc/hosts.equiv`, an attacker from any source address (no spoofing even needed) logs in or runs commands as the trusted account with no password.

```bash
# detect over-broad trust where the files are readable
grep -R '+' /etc/hosts.equiv ~*/.rhosts 2>/dev/null     # a bare + or "+ +" is wide open
# where present, access needs no password and often no spoofing
rlogin -l <user> <target>
rsh -l <user> <target> id
```

## Exploitation notes

- `+ +` and a bare `+` are the maximal cases: they trust everyone, so any reachable attacker gets passwordless access, no trusted-host control or spoofing required.
- A `+` only in the user field of a host entry trusts all users from that host; in the host field, all hosts for that user; read the fields carefully to see how wide the trust is.
- Netgroup (`+@group`) and wildcard host entries can resolve more broadly than intended; where you can read the file, evaluate what it actually matches.
- This is the "already open" case of trust abuse; where no wildcard exists, [plant one](rhosts-write.md) or [spoof a specific trusted host](ip-spoofing.md).

## References

- [man 5 rhosts (token semantics)](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
