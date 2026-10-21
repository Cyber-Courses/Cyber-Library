---
title: "Public key exposure: finding and reusing SSH private keys"
description: "An exposed SSH private key creates risk when the server trusts its public key and all applicable passphrase, source, expiry, certificate, user-verification, and multi-method requirements can be satisfied. Authorized-key options may further restrict the resulting session."
keywords:
  - ssh private key
  - authorized_keys
  - key reuse
  - passphrase
  - id_rsa
---

# Public key exposure

Private keys can be exposed through repositories, file shares, backups, image layers, automation systems, or incorrectly protected home directories. Possession alone does not guarantee access: the corresponding public key or signing authority must be trusted for the target account, a passphrase or hardware-backed user-verification requirement may apply, and the server may require another authentication method. `authorized_keys` options and certificates can also constrain source addresses, validity, commands, PTY allocation, and forwarding. Establish these conditions before describing the impact of a recovered key.

```bash
# hunt for private keys wherever files are readable
grep -rlE 'BEGIN (OPENSSH|RSA|EC|DSA) PRIVATE KEY' /mnt/share /loot 2>/dev/null
find / -name 'id_rsa' -o -name 'id_ed25519' -o -name '*.pem' -o -name '*.ppk' 2>/dev/null
# use a recovered key
chmod 600 id_rsa; ssh -i id_rsa user@<target>
# passphrase-protected: export a verifier for authorized offline candidate testing
ssh2john id_rsa > h && john --wordlist=rockyou.txt h
# convert PuTTY .ppk to OpenSSH if needed
puttygen key.ppk -O private-openssh -o id_rsa
# plant a key for access/persistence where you can write authorized_keys
echo 'ssh-ed25519 AAAA... a' >> ~victim/.ssh/authorized_keys
```

## Exploitation notes

- Search authorized data sources such as repositories and their history, file shares, backups, image layers, CI secret stores, and home directories.
- A passphrase-protected key permits offline candidate testing, but recovery depends on passphrase strength and the private-key derivation parameters.
- Map the public key to actual trust entries, accounts, certificates, and hosts, preserving options such as `from`, `command`, `expiry-time`, `restrict`, and `principals`.
- A writable `authorized_keys` path can create persistence only if ownership, permissions, `StrictModes`, account policy, and authentication configuration allow that file to be used.

## Tools

- [john the ripper (ssh2john)](https://github.com/openwall/john)
- [trufflehog (key discovery in repos/images)](https://github.com/trufflesecurity/trufflehog)

## References

- [OpenSSH authorized_keys](https://man.openbsd.org/sshd.8#AUTHORIZED_KEYS_FILE_FORMAT)
- [OpenSSH private-key format and passphrases](https://man.openbsd.org/ssh-keygen)
