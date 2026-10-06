---
title: "Trusted host bypass: passwordless rexec via host trust"
order: 3
description: "Where rexecd honours the host-based trust files, or where the related rsh path is reachable, an attacker executes commands with no password by connecting from, or spoofing, a trusted host, or by planting a .rhosts entry. This removes the credential requirement entirely, giving command execution as the target user."
keywords:
  - trusted host
  - rexec
  - rhosts
  - passwordless
  - command execution
---

# Trusted host bypass

Although rexec is password-based by design, the Berkeley trust model frequently overlaps: systems running rexec usually run rsh too, honouring `~/.rhosts` and `/etc/hosts.equiv`, and an attacker who establishes trust executes commands with no password through that trusted path. The techniques are the shared ones, connect from or spoof a trusted host, or plant a `.rhosts` entry where the target home is writable, after which command execution needs no credential. This sidesteps the cleartext password entirely and is the stronger route when trust can be obtained.

```bash
# if trust applies (or via the co-located rsh path), execute with no password
rsh -l <user> <target> id                      # trusted-host execution, no credential
# plant trust first where the home is writable
echo '+ +' >> ~victim/.rhosts
# then run commands as victim with no password
```

## Exploitation notes

- rexec hosts almost always also expose rsh/rlogin with the same trust files, so trust abuse established for one applies to all; the [rhosts](../rsh/rhosts-bypass.md) and [hosts.equiv](../trust-abuse/hosts-equiv-trust.md) techniques carry over directly.
- Trust removes the password requirement that rexec otherwise imposes, so obtaining trust is preferable to capturing or guessing the credential.
- Planting `.rhosts` needs a home-directory write; a permissive `hosts.equiv` or a spoofable trusted host needs no write at all.
- Where trust cannot be obtained, fall back to [capturing the cleartext password](cleartext-passwords.md).

## References

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
