---
title: "Known-hosts bypass: defeating SSH trust-on-first-use"
description: "SSH's protection against a substituted host key is the known_hosts pin and host-key verification. That protection can fail when a first-use fingerprint is accepted without independent verification, clients permit unknown or changed keys, or known_hosts can be modified, enabling machine-in-the-middle attacks."
keywords:
  - known_hosts
  - stricthostkeychecking
  - trust on first use
  - host key warning
  - tofu
---

# Known-hosts bypass

The pin in `known_hosts` is what stops an attacker substituting their own key. On a first connection there is no pin, so the default `StrictHostKeyChecking ask` prompts the user to confirm the presented fingerprint; accepting it without independent verification creates a trust-on-first-use opportunity. `accept-new` automatically records previously unseen keys but refuses changed keys, while `no` or `off` permits both new and changed keys, subject to OpenSSH's restrictions for changed keys. Where an attacker can write a victim's `known_hosts` (via a writable home over NFS/SMB, or a prior foothold), they can pre-pin their own key so no warning appears.

```bash
# conditions that let a substituted key be trusted:
grep -ri 'StrictHostKeyChecking' /etc/ssh/ssh_config ~/.ssh/config    # accept-new: new only; no/off: new and changed
# first-connection TOFU: a target that has never connected has no pin to violate
# pre-poison a victim's known_hosts where you can write it (attacker host key for the target)
ssh-keyscan -H <target> 2>/dev/null   # shows the format; an attacker writes THEIR key as the target's
```

## Exploitation notes

- The easiest wins are configuration and behaviour: `StrictHostKeyChecking accept-new` silently accepts an attacker's key for an unknown host, while `no` or `off` also allows a changed key to proceed subject to OpenSSH's restrictions.
- Trust on first use is exploitable when the client has not yet connected to the target and the user confirms the displayed fingerprint without verifying it through a trusted channel; `accept-new`, `no`, and `off` remove that confirmation for new keys.
- Writing a victim's `known_hosts` to pre-pin the attacker's key removes the warning entirely; combine with a home-directory write primitive (NFS UID spoof, writable share).
- A bypassed host-key check is the enabler for [SSH MITM](ssh-mitm.md); on its own it is the trust failure that makes interception possible.

## References

- [OpenSSH: StrictHostKeyChecking](https://man.openbsd.org/ssh_config#StrictHostKeyChecking)
- [HackTricks: SSH known_hosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
