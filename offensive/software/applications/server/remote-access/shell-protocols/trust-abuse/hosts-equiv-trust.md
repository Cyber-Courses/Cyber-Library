---
title: "hosts.equiv trust: exploiting system-wide r-command trust"
order: 3
description: "/etc/hosts.equiv defines system-wide host and user trust for rsh and rlogin. A host-only rule permits like-named accounts, while an explicit user or wildcard can grant broader passwordless access; root remains excluded."
keywords:
  - hosts.equiv
  - system-wide
  - trust
  - passwordless
  - all users
---

# hosts.equiv trust

`/etc/hosts.equiv` is the system-wide trust file used by `rsh` and `rlogin`. The effect depends on both fields. A host-only entry permits remote users to access like-named local accounts without a password. Adding a username permits that remote user to access non-root local accounts, while wildcard forms such as a bare `+` or `host +` broaden trust further. Abuse therefore depends on the exact rule: an attacker may control or spoof a listed host, match a trusted username, exploit a wildcard, or gain enough local privilege to modify the file. Root is excluded from `hosts.equiv` processing and instead relies on root's own `.rhosts` policy.

```bash
# read it to see which hosts are trusted and how broadly
cat /etc/hosts.equiv 2>/dev/null
# a host-only rule normally requires the remote and local usernames to match
rlogin -l <matching-user> <target>
rsh -l <matching-user> <target> id
# if writable after local elevation, this is a deliberately maximal rule
echo '+' >> /etc/hosts.equiv      # any host; behavior may also depend on PAM policy
```

## Exploitation notes

- Do not treat every entry as equivalent: `host` requires like-named accounts, whereas `host user`, `host +`, and a bare `+` have progressively broader consequences.
- Root is not covered by `hosts.equiv`; passwordless root requires `/.rhosts`, so note the distinction when targeting the root account.
- The file is root-owned, so writing it is a post-elevation/persistence step; the initial vector is usually an already-permissive file plus a trusted or [spoofable](ip-spoofing.md) source host.
- The file governs `rsh` and `rlogin`; it is not an authentication source for password-based `rexec`.

## References

- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
