---
title: "hosts.equiv: system-wide r-command trust abuse"
order: 3
description: "/etc/hosts.equiv defines system-wide host and user trust for rsh and rlogin. Host-only entries permit like-named accounts, while explicit users and wildcard rules can grant broader passwordless access; root is excluded."
keywords:
  - hosts.equiv
  - system-wide trust
  - rsh
  - passwordless
  - wildcard
---

# hosts.equiv

`/etc/hosts.equiv` defines system-wide trust for `rsh` and `rlogin`, but its breadth depends on the entry. A host-only rule permits remote users to access like-named local accounts without a password. A rule with a username permits that remote user to access non-root local accounts, and wildcard forms such as `host +` or a bare `+` are broader still. An attacker can abuse a host they control or can spoof when their username matches the rule, or modify the file after gaining elevated local access. Root is excluded from `hosts.equiv` processing.

```bash
# a host-only entry normally requires like-named remote and local users
rlogin -l <matching-user> <target>
rsh -l <matching-user> <target> id
# a wildcard/permissive hosts.equiv (for example, a bare "+" or "host +") trusts widely
# if writable (post-elevation), grant system-wide trust:
echo '+' >> /etc/hosts.equiv
```

## Exploitation notes

- A host-only entry is limited to like-named accounts; a username field or wildcard determines whether the same host can reach other non-root accounts.
- A bare `+` is the maximal weakness, although modern PAM configurations may require an explicit promiscuous option before treating it as a wildcard. Combined with [IP spoofing](ip-spoofing.md) of a trusted address, a matching rule can grant access without controlling the trusted host.
- Writing `hosts.equiv` needs elevated local access (it is root-owned), so it is more a persistence/escalation step than an initial vector; the initial vector is usually an already-permissive file plus a trusted or spoofed source.
- The same file governs `rsh` and `rlogin`, but not password-based `rexec`; see the [trust-abuse hosts.equiv page](../trust-abuse/hosts-equiv-trust.md) for the shared view.

## References

- [man 5 hosts.equiv](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [HackTricks: rsh](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
