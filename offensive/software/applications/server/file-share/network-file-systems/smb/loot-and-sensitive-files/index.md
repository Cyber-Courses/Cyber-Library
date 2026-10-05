---
title: "Loot and sensitive files: harvesting secrets from SMB shares"
description: "Harvesting secrets from readable SMB shares: credentials in scripts, configs, and SYSVOL Group Policy Preferences, and the backups, disk images, and password databases left on shares that contain data and credentials in bulk."
keywords:
  - SMB loot
  - credential hunting
  - GPP cpassword
  - backups
  - sensitive files
---

# Loot and sensitive files

Readable shares are where credentials and data leak at scale. Two veins are most productive: credentials scattered through scripts, configuration files, and SYSVOL Group Policy Preferences, and the bulk artifacts, backups, disk images, and password databases, that teams leave on shares and that contain everything at once.

## Subtopics

- **[Credential hunting](credential-hunting.md)**: passwords in scripts, configs, and SYSVOL.
- **[Backups and archives](backups-and-archives.md)**: backups, images, and password databases.

## References

- [NetExec: spidering and modules](https://www.netexec.wiki/smb-protocol/enumeration)
- [HackTricks: pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
