---
title: "rhosts write: planting a trusting .rhosts entry"
description: "Writing an entry into a target user's ~/.rhosts grants the attacker passwordless r-command access as that user and persists as a backdoor. The precondition is a write to the target's home directory, obtained through a writable NFS home, a file-write vulnerability, or an existing foothold as another user or root."
keywords:
  - rhosts write
  - .rhosts
  - backdoor
  - nfs home
  - persistence
---

# rhosts write

The most direct trust abuse is to write the trust yourself: adding a line to a target user's `~/.rhosts` grants the attacker passwordless rsh/rlogin/rexec access as that user, and the entry persists as a backdoor until removed. The whole attack reduces to obtaining a write into the target's home directory. That write comes from a writable home exported over NFS (where UID spoofing lets you write as the user), a file-write or path-traversal vulnerability in another service, or an existing foothold as another account or root. The entry `+ +` trusts everyone; a specific `host user` entry is stealthier.

```bash
# the precondition is a write to the target's home; common via NFS UID spoofing
mount -t nfs <target>:/home /mnt && sudo -u \#<victim_uid> sh -c \
  'echo "attacker-host attacker-user" >> /mnt/victim/.rhosts'
# or the maximal open-door entry
echo '+ +' >> /mnt/victim/.rhosts
# then access with no password
rlogin -l victim <target>
```

## Exploitation notes

- The enabling primitive is a home-directory write; the classic pairing is an [NFS export with AUTH_SYS](../../../../file-share/network-file-systems/nfs/uid-and-gid-spoofing.md), where spoofing the victim's UID lets you write their `.rhosts` as them.
- `+ +` is maximal (any host, any user) and obvious; a targeted `host user` entry trusting only your identity is quieter and still durable.
- This is both access and persistence: the planted trust grants repeated passwordless re-entry; include it when establishing a durable foothold on legacy Unix.
- Root's trust lives in `/.rhosts` (not covered by `hosts.equiv`); writing it grants passwordless root where you can reach that file.

## References

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: .rhosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
