---
title: "Wildcard trust: abusing + + and over-broad r-command entries"
order: 2
description: "A + token can act as a wildcard in .rhosts or hosts.equiv, making entries such as + + dangerously broad. Exact behavior is implementation- and PAM-policy-dependent, so testers must evaluate the service's trust parser rather than assume every bare + is active."
keywords:
  - wildcard
  - + +
  - rhosts
  - hosts.equiv
  - open trust
---

# Wildcard trust

Traditional r-command trust-file syntax uses `+` as an allow wildcard: `+` in the host field can match any host, `+` in the user field can match any remote user, and `+ +` is deliberately broad. Actual behavior depends on the implementation and authentication stack. In particular, modern PAM-based systems may treat a standalone `+` as a wildcard only when the service enables the `promiscuous` option. Rules are evaluated sequentially until the first match, so an earlier negative host, user, or netgroup rule may prevent a later wildcard from applying. A reviewer must therefore inspect the complete ordered ruleset and the service's trust configuration before concluding that passwordless access is open.

```bash
# detect over-broad trust where the files are readable
grep -R '+' /etc/hosts.equiv ~*/.rhosts 2>/dev/null
# inspect PAM/service policy, then test the exact host/user match
rlogin -l <user> <target>
rsh -l <user> <target> id
```

## Exploitation notes

- `+ +` and a bare `+` express maximally broad trust in traditional syntax, but the service may reject or constrain them through PAM or implementation-specific policy.
- A `+` only in the user field of a host entry trusts all users from that host; in the host field, all hosts for that user; read the fields carefully to see how wide the trust is.
- Netgroup (`+@group`) and wildcard host entries can resolve more broadly than intended; where you can read the file, evaluate what it actually matches.
- Preserve rule order during analysis: deny entries must appear before the allow rule they are intended to constrain because evaluation stops at the first match.
- This is the "already open" case of trust abuse; where no wildcard exists, [plant one](rhosts-write.md) or [spoof a specific trusted host](ip-spoofing.md).

## References

- [man 5 hosts.equiv (token and ordering semantics)](https://man7.org/linux/man-pages/man5/hosts.equiv.5.html)
- [Linux-PAM pam_rhosts](https://man7.org/linux/man-pages/man8/pam_rhosts.8.html)
- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
