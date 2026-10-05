---
title: "Loot and sensitive files: mining readable SMB shares"
description: "Readable SMB shares routinely hold the credentials and data that advance an intrusion: scripts and config files with embedded passwords, backups and disk images, Group Policy data in SYSVOL, and keys and tokens. Systematically spidering shares and pattern-matching their contents turns read access into credentials and sensitive data."
keywords:
  - smb loot
  - sensitive files
  - credential hunting
  - sysvol
  - spider
---

# Loot and sensitive files

A readable share is rarely just files; it is where an organization leaves the credentials and data that move an attacker forward. Scripts and configuration files embed service passwords and connection strings, backups and disk images contain whole systems to crack offline, the domain `SYSVOL` share holds Group Policy (and sometimes cached credentials), and developers and admins leave keys, tokens, and password lists in share directories. The method is to spider the readable shares and pattern-match their contents at scale rather than browsing by hand.

```bash
# inventory and search readable shares for likely loot
nxc smb <target> -u user -p 'pass' -M spider_plus -o DOWNLOAD_FLAG=True
# pattern-match downloaded content for secrets
grep -rinE 'password|passwd|pwd|secret|connectionstring|api[_-]?key' ./loot | head
# high-value filetypes to pull selectively
#   *.config *.ini *.xml *.ps1 *.bat *.vbs *.kdbx *.ppk *.pem id_rsa *.vmdk *.bak
```

## Subtopics

- **[Credential hunting](credential-hunting.md)**: finding passwords, keys, and tokens in share content.
- **[Backups and archives](backups-and-archives.md)**: mining backups, images, and archives for whole systems.

## References

- [NetExec: spidering shares](https://www.netexec.wiki/smb-protocol)
- [MITRE ATT&CK: unsecured credentials in files](https://attack.mitre.org/techniques/T1552/001/)
