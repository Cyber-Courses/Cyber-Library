---
title: "rhosts bypass: passwordless rlogin via host trust"
order: 4
description: "rlogin accepts a passwordless login when the target user's .rhosts or the system hosts.equiv trusts the connecting host and user. Abusing existing trust, or planting a .rhosts entry where the home directory is writable, grants an interactive shell as the target with no password, the same host-trust weakness as rsh applied to interactive login."
keywords:
  - rhosts
  - rlogin
  - passwordless
  - trusted host
  - interactive shell
---

# rhosts bypass

`rlogin` honours the same host-based trust as rsh: if the target user's `~/.rhosts` or the system `/etc/hosts.equiv` trusts the connecting host and username, the login succeeds with no password, dropping straight to an interactive shell. The attack is identical in mechanism to the rsh case but yields a login session rather than a single command. An attacker uses existing trust (connecting from, or spoofing, a trusted host), or plants a `.rhosts` entry where they can write the target's home, then `rlogin -l <user>` for a passwordless interactive shell.

```bash
# passwordless interactive login when trusted
rlogin -l <user> <target>
# plant trust first if you can write the target's home (e.g. over NFS)
echo 'attacker-host attacker-user' >> ~victim/.rhosts
rlogin -l victim <target>          # now no password
```

## Exploitation notes

- The payoff versus rsh is an interactive shell (rlogin) rather than one command, so it is preferred when you want a session; the trust mechanism and the ways to obtain it are the same.
- Planting `.rhosts` needs a home-directory write (NFS UID spoof, writable share, foothold); `+ +` opens the account to any host/user, a targeted entry is stealthier, see [rhosts write](../trust-abuse/rhosts-write.md).
- Trust applies per user, so target a specific account's `.rhosts`; for breadth, a permissive [hosts.equiv](../trust-abuse/hosts-equiv-trust.md) covers all non-root users.
- Where trust is not available, rlogin falls back to a password you can [capture in cleartext](cleartext-passwords.md).

## References

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: rlogin](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rlogin)
