---
title: "Restricted shell and chroot escape: breaking out of an SFTP jail"
description: "Escaping a restricted SFTP-only account confined by an OpenSSH internal-sftp chroot or a ForceCommand, to reach a full shell or files outside the jail, by abusing writable paths, misconfigured chroot ownership, or features like ProxyJump and local port forwarding the server still permits."
keywords:
  - SFTP chroot
  - internal-sftp
  - ForceCommand
  - restricted shell
  - jailbreak
---

# Restricted shell and chroot escape

Admins confine SFTP users with `ForceCommand internal-sftp` and a `ChrootDirectory`, intending a file-only jail. The confinement leaks when the chroot is misconfigured (writable by the user, or wrong ownership breaks the chroot entirely), when the account can still open SSH channels the config did not disable (port forwarding, ProxyJump), or when a writable path inside the chroot maps to execution outside it.

```bash
# Can the "SFTP-only" account still forward or tunnel?
ssh -N -L 8080:127.0.0.1:80 user@<target>     # local forward, if not disabled
ssh -J user@<target> internal-host             # ProxyJump through the jail
# Writable chroot: upload to a path the host executes (cron, web root bound in)
```

## Exploitation notes

- OpenSSH requires the ChrootDirectory and its path to be root-owned and not writable; a violation disables the chroot, exposing the full filesystem over SFTP.
- Even a correct SFTP chroot does not restrict SSH port forwarding unless `AllowTcpForwarding no` and `PermitTunnel no` are set, so tunneling into the internal network often still works.
- A writable directory inside the chroot that is bind-mounted from a host-executed path (web root, cron) turns file write into code execution outside the jail.

## References

- [OpenSSH sshd_config: ChrootDirectory](https://man.openbsd.org/sshd_config#ChrootDirectory)
- [HackTricks: pentesting SSH](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ssh.html)
