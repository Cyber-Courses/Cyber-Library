---
title: "rhosts write: planting a trusting .rhosts entry"
order: 4
description: "Writing an entry into a target user's ~/.rhosts grants the attacker passwordless r-command access as that user and persists as a backdoor. The precondition is a write to the target's home directory, obtained through a writable NFS home, a file-write vulnerability, or an existing foothold as another user or root."
keywords:
  - rhosts write
  - .rhosts
  - backdoor
  - nfs home
  - persistence
---

# rhosts write

The most direct trust abuse is to write the trust yourself: adding a valid line to a target user's `~/.rhosts` can grant passwordless `rsh` or `rlogin` access as that user, and the entry persists until removed. The attacker needs a write that preserves the trust file's acceptance conditions. Common `ruserok` implementations reject a file that is not regular, is owned by anyone other than the target user or root, is writable by anyone other than its owner, or has another hard link. A writable NFS home with matching AUTH_SYS identity can satisfy those requirements; a generic file-write, path traversal, or foothold as another account works only if it can create or modify an acceptable file. The entry `+ +` trusts everyone where wildcard policy permits it; a specific `host user` entry is narrower. This trust does not authenticate `rexec`, which uses a username and password.

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

- The enabling primitive is a home-directory write; the classic pairing is an [NFS export with AUTH_SYS](../../../file-share/network-file-systems/nfs/uid-and-gid-spoofing.md), where spoofing the victim's UID lets you write their `.rhosts` as them.
- Confirm that `.rhosts` is a regular file owned by the target user or root, is not writable by other users, and meets the implementation's hard-link policy; content alone is insufficient if the trust-file safety checks fail.
- `+ +` is maximal (any host, any user) and obvious; a targeted `host user` entry trusting only your identity is quieter and still durable.
- This is both access and persistence: the planted trust grants repeated passwordless re-entry; include it when establishing a durable foothold on legacy Unix.
- Root's trust lives in `/.rhosts` (not covered by `hosts.equiv`); writing it grants passwordless root where you can reach that file.

## References

- [Linux rcmd/ruserok authentication routines](https://man7.org/linux/man-pages/man3/rcmd.3.html)
- [Linux-PAM pam_rhosts](https://man7.org/linux/man-pages/man8/pam_rhosts.8.html)
- [HackTricks: .rhosts](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
