---
title: "Backups and archives: mining stored systems for offline compromise"
description: "Shares used as backup targets hold whole systems: VM disk images, database dumps, system-state and registry backups, mailbox exports, and archives. Read access to these lets an attacker reconstruct systems offline, extracting password hashes from registry hives, secrets from database dumps, and files from images, without touching the live systems they came from."
keywords:
  - backups
  - vmdk
  - registry hive
  - database dump
  - archive
---

# Backups and archives

Backup shares are among the richest loot because a backup is a complete copy of a system, readable offline with none of the live system's defenses. A readable backup target yields VM disk images, system-state and registry backups, database dumps, mailbox exports, and general archives, each reconstructable into credentials and data. The method is to identify the backup artefacts, pull them, and extract offline.

## Identify and pull

```bash
# backup artefacts commonly found on backup shares
find ./loot -iregex '.*\.\(bak\|vhd\|vhdx\|vmdk\|vbk\|ova\|tar\|zip\|7z\|gz\|bkf\|wbcat\)$'
#   plus: *.ldf/*.mdf (SQL), *.bacpac/*.dacpac, *.pst/*.ost (mailboxes),
#         ntds.dit, SYSTEM/SECURITY/SAM hives, System State backups
```

## Extract credentials and data offline

```bash
# Windows registry hives from a backup -> local account hashes and LSA secrets
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL
# a domain controller backup containing ntds.dit -> every domain hash
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
# VM disk images -> mount and read like any disk theft
guestmount -a system.vmdk -i --ro /mnt && ls /mnt/Windows/System32/config
# database dumps -> restore or grep for secrets and data
strings backup.bak | grep -iE 'password|secret|connectionstring' | head
```

A backup of a domain controller that includes `ntds.dit` plus the `SYSTEM` hive is the jackpot: `secretsdump` extracts every domain account's hash offline, which is domain-wide compromise from a single readable archive.

## Exploitation notes

- The `ntds.dit` + `SYSTEM` combination from any DC backup yields all domain hashes offline; prioritise backup shares and system-state backups for it.
- Local hive sets (`SAM` + `SYSTEM` + `SECURITY`) give local admin hashes and LSA secrets (which often include service-account passwords in cleartext-equivalent form).
- VM images and `.vbk`/`.vmdk` files are read with the same offline-mount techniques as hypervisor disk theft; prioritise credential stores inside.
- Everything here is offline and touches no live system, so it is low-noise; the constraint is only read access to the backup share.

## Tools

- [Impacket secretsdump](https://github.com/fortra/impacket)
- [libguestfs / guestmount](https://libguestfs.org/)

## References

- [MITRE ATT&CK: data from backups](https://attack.mitre.org/techniques/T1530/)
- [Impacket secretsdump usage](https://github.com/fortra/impacket)
