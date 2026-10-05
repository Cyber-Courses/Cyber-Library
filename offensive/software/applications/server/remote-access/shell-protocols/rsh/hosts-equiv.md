---
title: "hosts.equiv: system-wide r-command trust abuse"
description: "/etc/hosts.equiv grants passwordless r-command access system-wide for the listed trusted hosts, applying to all non-root users. A permissive or attacker-modified hosts.equiv, especially a wildcard entry, lets an attacker from a trusted (or spoofed) host log in as any user without a password, a broader trust than per-user .rhosts."
keywords:
  - hosts.equiv
  - system-wide trust
  - rsh
  - passwordless
  - wildcard
---

# hosts.equiv

`/etc/hosts.equiv` is the system-wide equivalent of `.rhosts`: it lists hosts (and optionally users) trusted for passwordless r-command access, and it applies to all non-root accounts on the system at once. That breadth makes it more dangerous than a single user's `.rhosts`. A permissive `hosts.equiv`, one listing a host the attacker controls or can spoof, or a wildcard entry, lets the attacker log in as essentially any user from that host with no password. If the attacker can write `hosts.equiv` (requiring elevated local access), they grant themselves system-wide trust directly.

```bash
# if a trusted host is attacker-controlled or spoofable, log in as any user
rlogin -l <anyuser> <target>          # accepted if the source host is in hosts.equiv
rsh -l <anyuser> <target> id
# a wildcard/permissive hosts.equiv (e.g. a bare "+" or a broad host entry) trusts widely
# if writable (post-elevation), grant system-wide trust:
echo '+' >> /etc/hosts.equiv
```

## Exploitation notes

- `hosts.equiv` applies to all non-root users, so one permissive entry exposes every account on the host; it does not cover root (root uses only `/.rhosts`), a common point of confusion.
- A bare `+` or an overly broad host entry is the maximal weakness; combined with [IP spoofing](ip-spoofing.md) of a trusted address, it grants access without controlling the trusted host.
- Writing `hosts.equiv` needs elevated local access (it is root-owned), so it is more a persistence/escalation step than an initial vector; the initial vector is usually an already-permissive file plus a trusted or spoofed source.
- The same file governs rsh, rlogin, and rexec; see the [trust-abuse hosts.equiv page](../trust-abuse/hosts-equiv-trust.md) for the cross-command view.

## References

- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
