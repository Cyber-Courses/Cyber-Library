---
title: "hosts.equiv trust: exploiting system-wide r-command trust"
description: "/etc/hosts.equiv grants passwordless r-command access for trusted hosts to all non-root users at once, a broader trust than per-user .rhosts. A permissive entry, a trusted host an attacker controls or can spoof, or write access to the file (post-elevation) gives passwordless access as any non-root account on the system."
keywords:
  - hosts.equiv
  - system-wide
  - trust
  - passwordless
  - all users
---

# hosts.equiv trust

`/etc/hosts.equiv` is the system-wide trust file for the r-commands: hosts (and optionally users) listed there are trusted for passwordless access to all non-root accounts on the system simultaneously. That makes it broader and more dangerous than a single user's `.rhosts`. The abuse cases: the file already lists a host the attacker controls or can spoof (passwordless access as any non-root user from there), it contains an over-broad or wildcard entry, or the attacker has gained enough local privilege to write it and grant themselves system-wide trust. Root is excepted, root trust lives only in `/.rhosts`.

```bash
# read it to see which hosts are trusted and how broadly
cat /etc/hosts.equiv 2>/dev/null
# from a trusted (or spoofed) host, log in as any non-root user with no password
rlogin -l <anyuser> <target>
rsh -l <anyuser> <target> id
# if writable after local elevation, grant system-wide trust
echo '+' >> /etc/hosts.equiv      # trust all hosts (open door for all non-root users)
```

## Exploitation notes

- The breadth is the point: one trusted entry exposes every non-root account, so `hosts.equiv` trust is a wider win than per-user [.rhosts](rhosts-write.md); enumerate which users exist to pick a useful target account.
- Root is not covered by `hosts.equiv`; passwordless root requires `/.rhosts`, so note the distinction when targeting the root account.
- The file is root-owned, so writing it is a post-elevation/persistence step; the initial vector is usually an already-permissive file plus a trusted or [spoofable](ip-spoofing.md) source host.
- Applies uniformly to rsh, rlogin, and rexec on the host.

## References

- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
