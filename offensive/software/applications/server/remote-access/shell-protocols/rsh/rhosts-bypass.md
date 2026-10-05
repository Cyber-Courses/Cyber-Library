---
title: "rhosts bypass: passwordless rsh via .rhosts trust"
description: "The r-commands grant passwordless access when the target user's ~/.rhosts file lists the connecting host and user as trusted. An attacker who can write that file, through another vulnerability, a writable home over NFS, or an existing foothold, adds a trusting entry and then rsh/rlogin in as that user with no password, which is also a durable backdoor."
keywords:
  - rhosts
  - passwordless
  - trusted host
  - rsh
  - backdoor
---

# rhosts bypass

Each user can grant passwordless r-command access by listing trusted `host user` pairs in their `~/.rhosts` file; a matching connection is accepted with no password. The offensive use is to create or extend that trust. If an attacker can write a target user's `.rhosts`, via a file-write vulnerability, a writable home directory (commonly over NFS), or an existing foothold as another user, they add an entry trusting their own host and username, then `rsh`/`rlogin` in as the target with no credential. The wildcard entry `+ +` trusts every host and user, turning the account into an open door. This doubles as persistence.

```bash
# plant trust in the target user's .rhosts (requires a write to their home)
echo '+ +' >> /mnt/nfs-home/victim/.rhosts            # trust everyone (open door)
echo 'attacker-host attacker-user' >> ~victim/.rhosts  # trust a specific identity
# then log in / run commands with no password
rlogin -l victim <target>
rsh -l victim <target> id
```

## Exploitation notes

- The precondition is a write to the target's `.rhosts`; pair with any home-directory write primitive (an [NFS UID-spoof write](../../../../file-share/network-file-systems/nfs/uid-and-gid-spoofing.md), a writable share, or a foothold), which is exactly why writable home directories are dangerous with r-commands enabled.
- `+ +` is the maximal entry (any host, any user); a targeted `host user` entry is stealthier.
- This is both access and persistence: the planted trust survives and grants passwordless re-entry until the file is cleaned.
- The same `.rhosts` mechanism serves rsh, rlogin, and rexec; see [rhosts write](../trust-abuse/rhosts-write.md) for the write technique in the trust-abuse view.

## References

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: rsh/.rhosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
