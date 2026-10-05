---
title: "Username enumeration: discovering valid SFTP and SSH accounts"
description: "SSH can leak which usernames are valid through authentication-method differences and, on vulnerable versions, timing side channels. A validated user list focuses password spraying and key attacks against real accounts, and distinguishes SFTP-only accounts from full-shell accounts, which shapes the follow-on from transfer access to command execution."
keywords:
  - username enumeration
  - ssh
  - auth methods
  - timing
  - user list
---

# Username enumeration

Before attacking credentials, knowing which usernames exist focuses the effort and avoids noise against non-existent accounts. SSH can leak this in a few ways: the advertised authentication methods sometimes differ between valid and invalid users, certain OpenSSH versions had a timing side channel where a valid user took measurably longer to reject, and the banner or allowed methods can distinguish account types. A validated list then drives spraying and key attacks against real targets and helps separate SFTP-only accounts from full-shell ones.

```bash
# which auth methods the server offers (and whether they vary per user)
ssh -o PreferredAuthentications=none -o PubkeyAuthentication=no user@<target>
nmap -p22 --script ssh-auth-methods --script-args="ssh.user=<user>" <target>
# timing-based enumeration on vulnerable OpenSSH (valid users slower to reject)
#   send a malformed/oversized auth and compare response timing across candidates
```

## Exploitation notes

- The most reliable modern signal is differing accepted authentication methods per user (for example, a user configured for keys only versus password), which `ssh-auth-methods` surfaces; the timing side channel applies only to specific older OpenSSH versions.
- Seed candidate usernames from other enumeration (SMB RID cycling, web app accounts, OSINT, common service names) and validate here before spraying.
- Distinguishing SFTP-only accounts (confined to the subsystem) from shell accounts matters: the former lead to the [restricted-shell escape](restricted-shell-and-chroot-escape.md), the latter directly to command execution once authenticated.
- Keep it low-noise; enumeration that triggers auth failures is logged, so prefer the method-difference check over mass timing probes where possible.

## References

- [nmap ssh-auth-methods](https://nmap.org/nsedoc/scripts/ssh-auth-methods.html)
- [HackTricks: SSH user enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
