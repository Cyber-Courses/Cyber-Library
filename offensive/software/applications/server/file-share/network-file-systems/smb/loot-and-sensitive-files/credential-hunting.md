---
title: "Credential hunting: finding passwords across SMB shares"
description: "Finding credentials across readable SMB shares: passwords in scripts, config and unattend files, connection strings, and the famous Group Policy Preferences cpassword in SYSVOL, then validating the recovered credentials against the domain."
keywords:
  - credential hunting
  - GPP cpassword
  - SYSVOL
  - unattend.xml
  - connection strings
---

# Credential hunting

Shares accumulate credentials: deployment scripts with embedded passwords, application configs with connection strings, `unattend.xml` and `sysprep` files from imaging, and the Group Policy Preferences `cpassword` in SYSVOL, which is encrypted with a published static key and so trivially decrypted. Sweeping shares for these patterns yields working credentials.

```bash
# SYSVOL GPP cpassword (readable by any domain user)
nxc smb <dc> -u user -p pass -M gpp_password
# Spider and grep shares for secrets
nxc smb <target> -u user -p pass -M spider_plus
grep -rniE 'password|pwd|connectionstring|api[_-]?key|secret' /mnt/share
```

## Exploitation notes

- GPP `cpassword` is the classic win: readable in SYSVOL by any domain user and decryptable with a known key, often yielding a privileged service account.
- `unattend.xml`, `sysprep.inf`, and `web.config` commonly hold local admin or service credentials in plaintext or weakly encoded.
- Validate recovered credentials broadly (`nxc smb <range> -u u -p p`) to find where they are reused.

## References

- [NetExec: gpp_password module](https://www.netexec.wiki/smb-protocol/modules)
- [HackTricks: GPP / cPassword](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)
