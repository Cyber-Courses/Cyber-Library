---
title: "Credential brute force: guessing SFTP logins"
description: "Brute-forcing SFTP credentials against the underlying SSH service, spraying common and default passwords, mindful that SFTP accounts are often service or application identities with weak, static passwords."
keywords:
  - SFTP brute force
  - SSH brute force
  - hydra
  - password spray
  - weak passwords
---

# Credential brute force

SFTP authenticates through SSH, so brute force targets the SSH service. SFTP-only accounts are frequently service identities (application file drops, partner exchanges) with weak, rarely-rotated passwords, which makes spraying productive. SSH may rate-limit or lock, so pace the attempts.

```bash
hydra -L users.txt -P passwords.txt sftp://<target>
nxc ssh <target> -u users.txt -p passwords.txt        # also validates SFTP access
```

## Exploitation notes

- Prefer spraying one password across many users over hammering one account, to avoid lockout and detection.
- SFTP service accounts for B2B file exchange often have guessable names and vendor-default passwords.
- A valid SFTP login may be SFTP-only; turning it into a shell is [Restricted shell and chroot escape](restricted-shell-and-chroot-escape.md).

## References

- [HackTricks: pentesting SSH](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ssh.html)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
