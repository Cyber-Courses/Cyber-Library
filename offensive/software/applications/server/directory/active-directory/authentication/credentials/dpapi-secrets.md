---
title: "DPAPI secrets: browser, vault, and application credentials"
order: 3
description: "Recovering credentials protected by the Windows Data Protection API: saved browser passwords, Windows Credential Manager vaults, RDP and scheduled-task credentials, by decrypting DPAPI blobs with the user or domain master key."
keywords:
  - DPAPI
  - master key
  - credential manager
  - browser passwords
  - dpapi backup key
---

# DPAPI secrets

The Data Protection API (DPAPI) is how Windows encrypts per-user secrets at rest: saved browser passwords, Windows Credential Manager entries, RDP saved credentials, scheduled-task passwords, and application secrets. Each is a DPAPI blob decryptable only with the user's **master key**, which is itself derived from the user's password (or recoverable by the domain). Reaching these secrets converts a foothold into a trove of a user's other passwords.

## The decryption chain

A DPAPI blob is decrypted with a master key; the master key is unlocked in one of three ways:

- **The user's password or NTLM hash** (you have it, or you are running as them).
- **The master key already cached in memory** (Mimikatz `sekurlsa::dpapi` pulls decrypted master keys from LSASS).
- **The domain DPAPI backup key**, a forest-wide private key held on the domain controllers. With it, Domain Admin can decrypt **any** user's DPAPI secrets in the domain, offline and forever.

```
# Extract the domain backup key once (Domain Admin), then decrypt anyone's blobs
mimikatz: lsadump::backupkeys /system:<dc> /export
dpapi.py backupkeys -t EXAMPLE/admin:password@<dc>
```

## Recovering the secrets

```
# Mimikatz on-host: master keys then the vaulted/credential blobs
sekurlsa::dpapi
dpapi::cred /in:"%APPDATA%\Microsoft\Credentials\<guid>"
vault::cred /patch

# Impacket, with a recovered master key
dpapi.py masterkey -file <masterkey> -sid <user-sid> -password <user-pass>
dpapi.py credential -file <cred-blob> -key <decrypted-masterkey>
```

Browser credential stores (Chrome/Edge `Login Data`, cookies) are DPAPI-protected on older versions; recent Chromium adds app-bound encryption, so a current browser needs an additional step beyond the DPAPI master key.

## Exploitation notes

- The domain backup key is the high-value target: one extraction (requires Domain Admin or the DC) permanently enables decrypting every domain user's DPAPI secrets, so it is both a credential-harvesting and a persistence primitive.
- DPAPI frequently yields credentials for *other* systems (saved RDP, VPN, app logins) that are not otherwise recoverable from hashes, widening lateral movement beyond the AD account set.
- Scheduled-task and service credentials stored via Credential Manager often belong to privileged automation accounts.

## Tools

- **Mimikatz** (`sekurlsa::dpapi`, `dpapi::cred`, `lsadump::backupkeys`): on-host decryption and backup-key export.
- **Impacket dpapi.py**: offline master-key and blob decryption, backup-key extraction.
- **SharpDPAPI / DonPAPI**: automated collection and decryption across hosts.

## References

- [DonPAPI (login-securite): automated DPAPI looting](https://github.com/login-securite/DonPAPI)
- [SharpDPAPI (GhostPack)](https://github.com/GhostPack/SharpDPAPI)
- [Impacket dpapi.py (fortra)](https://github.com/fortra/impacket)
- [Microsoft: Windows Data Protection (DPAPI)](https://learn.microsoft.com/en-us/previous-versions/ms995355%28v=msdn.10%29)
