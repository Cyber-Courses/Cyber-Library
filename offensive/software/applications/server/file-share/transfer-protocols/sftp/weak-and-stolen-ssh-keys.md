---
title: "Weak and stolen SSH keys: key-based SFTP access"
description: "Gaining SFTP access through SSH keys: reusing private keys recovered from shares, backups, or other hosts, abusing keys without passphrases, and the historically weak keys from flawed key generation, authenticating as the key's user without a password."
keywords:
  - SSH key
  - private key reuse
  - authorized_keys
  - weak key
  - passphrase
---

# Weak and stolen SSH keys

SFTP often authenticates with SSH keys rather than passwords. A private key recovered from a share, backup, home directory, or another compromised host authenticates as its owner wherever the matching public key is authorized, frequently without a passphrase. Historically, flawed key generation also produced guessable keys.

```bash
# Use a recovered private key for SFTP
chmod 600 id_rsa && sftp -i id_rsa user@<target>
# Passphrase-protected keys: crack offline
ssh2john id_rsa > hash && john hash
```

## Exploitation notes

- Private keys spread widely: developer home directories, CI configs, backups, and `.ssh` folders on shares; one key often works on many hosts.
- Keys without a passphrase are immediate access; passphrase-protected keys are cracked offline with ssh2john and a wordlist.
- Reuse is the theme: test a recovered key broadly, as teams share and copy keys across servers.

## References

- [HackTricks: pentesting SSH](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ssh.html)
- [OpenSSH key management](https://www.openssh.com/manual.html)
