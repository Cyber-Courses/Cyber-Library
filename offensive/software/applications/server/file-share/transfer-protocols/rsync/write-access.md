---
title: "Write access: planting files through a writable rsync module"
description: "An rsync module configured read-write (read only = false) lets an attacker upload files to the server directory it maps. Where that directory is consumed by the host, a webroot, a home directory, a cron or scripts path, the write becomes code execution or credential planting, bounded by the daemon's chroot, uid, and whether it preserves permissions."
keywords:
  - rsync write
  - read only false
  - upload
  - webroot
  - authorized_keys
---

# Write access

An rsync module with `read only = false` accepts uploads, so an attacker writes files into the server directory it maps. As with any writable share, the impact depends on what consumes that directory: a module mapping a webroot turns an upload into a web shell, one mapping a home directory lets you plant `authorized_keys`, and one mapping a cron or scripts path runs your code on the host's schedule. The daemon's configuration, chroot (`use chroot`), the `uid`/`gid` it runs writes as, and whether it preserves permissions, bounds how far the write reaches and what ownership the planted files take.

```bash
# confirm the module is writable
echo test > t; rsync -av t rsync://<target>/<module>/ && echo "writable"
# plant into a consumed location (examples depend on what the module maps)
rsync -av shell.php rsync://<target>/<webroot-module>/        # web shell if it is a webroot
rsync -av --chmod=600 authorized_keys rsync://<target>/<home-module>/.ssh/   # key plant
rsync -av cronjob rsync://<target>/<cron-module>/             # if it maps /etc/cron.d
```

## Exploitation notes

- Writability alone is not execution; the payoff requires the mapped directory to be acted on, so identify what the module maps (webroot, home, cron, deploy) from its name, contents, and server behaviour.
- The daemon's `uid`/`gid` determine the planted file's ownership and thus which consumers accept it; a daemon running as root (or writing to a root-consumed path) makes `authorized_keys`/cron plants effective.
- `use chroot` confines writes to the module root; without additional misconfiguration you cannot traverse out, so target what is inside the module that the host still executes.
- Prefer a deterministic consumer (cron, a login key, a served webroot) over hoping a file is run; the mechanics mirror [writable SMB share poisoning](../../network-file-systems/smb/writable-share-poisoning/index.md).

## References

- [rsyncd.conf: read only, use chroot, uid/gid](https://download.samba.org/pub/rsync/rsyncd.conf.5)
- [HackTricks: rsync write](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync)
