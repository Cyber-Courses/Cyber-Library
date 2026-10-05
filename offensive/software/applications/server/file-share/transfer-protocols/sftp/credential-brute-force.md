---
title: "Credential brute force: password attacks over SSH for SFTP"
description: "SFTP accounts authenticate through SSH, so password brute force and spraying go against the SSH service. Valid credentials give SFTP file access and, where the account also has a shell, direct command execution. SSH rate limits and key-only configurations shape the attack, favouring low-and-slow spraying of validated usernames with likely passwords."
keywords:
  - ssh brute force
  - password spray
  - sftp
  - hydra
  - credential reuse
---

# Credential brute force

Because SFTP authentication is SSH authentication, password attacks target the SSH service on port 22. A valid password gives SFTP file access, and if the account is not restricted to the SFTP subsystem, it gives a full shell, so credential guessing can land command execution directly. SSH servers often throttle attempts and may be key-only (no password auth), so the attack is best run as a low-and-slow spray of validated usernames against likely passwords rather than a fast brute force.

```bash
# confirm password auth is even offered before spraying
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@<target>
# spray one likely password across validated users (quiet), stop on success
hydra -L users.txt -p 'Summer2025!' ssh://<target> -t 4 -f
nxc ssh <target> -u users.txt -p passwords.txt          # netexec, marks shell vs no-shell
# a single credential: test both subsystem and shell
sftp user@<target>; ssh user@<target> id
```

## Exploitation notes

- Check that password authentication is enabled first; many SSH servers are key-only, in which case pivot to the [key](weak-and-stolen-ssh-keys.md) routes rather than guessing passwords.
- Spray slowly and with few threads: SSH logs every failure and may rate-limit or block, so a validated user list plus a short, context-derived password list beats a large wordlist.
- NetExec marks whether a working credential yields a shell or only the SFTP subsystem; a shell is immediate execution, an SFTP-only account routes to the [restricted-shell escape](restricted-shell-and-chroot-escape.md).
- SSH credentials are prime reuse material; test any hit against other hosts and services.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec (ssh)](https://github.com/Pennyw0rth/NetExec)

## References

- [HackTricks: SSH brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [OpenSSH sshd_config auth options](https://man.openbsd.org/sshd_config)
