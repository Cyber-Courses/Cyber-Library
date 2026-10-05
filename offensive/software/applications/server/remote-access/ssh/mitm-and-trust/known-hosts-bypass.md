---
title: "Known-hosts bypass: defeating SSH trust-on-first-use"
description: "SSH's protection against a substituted host key is the known_hosts pin and the client's host-key-changed warning. That protection fails on first connection (no pinned key), when clients are configured with StrictHostKeyChecking no, when users accept the warning, and when known_hosts can be modified, each of which lets an attacker's host key be trusted and enables machine-in-the-middle."
keywords:
  - known_hosts
  - stricthostkeychecking
  - trust on first use
  - host key warning
  - tofu
---

# Known-hosts bypass

The pin in `known_hosts` plus the loud warning on a changed host key is what stops an attacker substituting their own key. That trust model has gaps an attacker exploits to get their host key accepted. On a first-ever connection there is no pinned key, so trust-on-first-use accepts whatever is presented. Many clients and automation set `StrictHostKeyChecking no` (or `accept-new`), silently accepting new keys. Users habituated to the warning type `yes`. And where an attacker can write a victim's `known_hosts` (via a writable home over NFS/SMB, or a prior foothold), they pre-pin their own key so no warning ever appears.

```bash
# conditions that let a substituted key be trusted:
grep -ri 'StrictHostKeyChecking' /etc/ssh/ssh_config ~/.ssh/config    # "no"/"accept-new" => silent accept
# first-connection TOFU: a target that has never connected has no pin to violate
# pre-poison a victim's known_hosts where you can write it (attacker host key for the target)
ssh-keyscan -H <target> 2>/dev/null   # shows the format; an attacker writes THEIR key as the target's
```

## Exploitation notes

- The easiest wins are configuration and behaviour: `StrictHostKeyChecking no` in a client/automation config means a MITM host key is accepted silently, and automation (CI, scripts) very commonly sets this.
- Trust-on-first-use is exploitable whenever the client has not yet connected to the target; being on-path for that first connection captures it with only a one-time warning (or none, if strict checking is off).
- Writing a victim's `known_hosts` to pre-pin the attacker's key removes the warning entirely; combine with a home-directory write primitive (NFS UID spoof, writable share).
- A bypassed host-key check is the enabler for [SSH MITM](ssh-mitm.md); on its own it is the trust failure that makes interception possible.

## References

- [OpenSSH: StrictHostKeyChecking](https://man.openbsd.org/ssh_config#StrictHostKeyChecking)
- [HackTricks: SSH known_hosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
