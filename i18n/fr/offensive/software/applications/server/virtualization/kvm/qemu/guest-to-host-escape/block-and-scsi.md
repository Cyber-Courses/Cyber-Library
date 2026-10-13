---
order: 4
title: "Bloc et SCSI : s'évader de QEMU via les contrôleurs de stockage"
description: "Le chemin de stockage de QEMU s'étend des contrôleurs IDE et AHCI émulés aux périphériques paravirtuels virtio-blk et virtio-scsi. L'invité programme des structures de commande et des descripteurs DMA que QEMU lit pour effectuer les E/S disque, si bien que des failles dans l'analyse des commandes, dans la gestion de la liste de commandes et des PRD AHCI, et dans le traitement des requêtes SCSI donnent un accès hors limites dans le processus QEMU."
keywords:
  - ahci
  - ide
  - virtio-blk
  - virtio-scsi
  - prd
---

# Bloc et SCSI

Le disque d'un invité QEMU est servi par un contrôleur de stockage émulé : IDE legacy, le contrôleur SATA AHCI, ou les périphériques paravirtuels virtio-blk et virtio-scsi. Chacun prend des structures de commande et des descripteurs DMA programmés par l'invité et les lit pour effectuer les E/S. AHCI utilise une liste de commandes et des tables de descripteurs de région physique (PRD) en mémoire invité ; les chemins SCSI analysent les blocs de descripteur de commande et les longueurs de transfert ; virtio-blk/scsi utilisent le modèle de descripteurs de virtqueue. L'analyse de ces structures contrôlées par l'invité dans QEMU est la surface d'évasion, la gestion de la liste de commandes/PRD AHCI et l'analyse des requêtes SCSI étant les lieux récurrents.

## La surface

```c
// AHCI: the guest sets the command list base; each command header points at a
// command table with a PRD (physical region descriptor) table. QEMU walks the PRDs,
// each giving a guest address and byte count, to DMA data. Primitives:
//  - a PRD count/length QEMU trusts for a transfer -> OOB read/write
//  - a command-list/command-table offset the model uses without bounding
// SCSI (scsi-disk / virtio-scsi): a CDB and transfer length; a length used for a
//  buffer operation beyond the allocation, or an unexpected command, -> OOB
// virtio-blk/scsi: request headers over the virtqueue (see virtio devices)
```

```bash
lspci -nn | grep -iE 'sata|ahci|scsi|virtio'   # which storage controller
```

## Notes d'exploitation

- La gestion des PRD AHCI est une cible classique : l'invité contrôle entièrement la table PRD (adresses et comptes) que QEMU utilise pour le DMA, si bien qu'un compte ou une longueur en lequel il se fie est une primitive de transfert hors limites directe.
- L'analyse des requêtes SCSI (partagée par les backends scsi-disk et virtio-scsi) traite des blocs de commande et des longueurs de transfert choisis par l'invité, un autre endroit où les hypothèses de longueur se brisent.
- virtio-blk/scsi passent par la couche virtqueue partagée ; l'analyse spécifique au stockage est l'en-tête de requête, voir [périphériques virtio](virtio-devices.md).
- L'invité programme ces structures directement depuis son pilote de stockage ; les primitives atterrissent dans QEMU, spécifiques à une version de QEMU.

## Références

- [Documentation stockage QEMU](https://www.qemu.org/docs/master/system/devices/)
- [Spécification AHCI (SATA)](https://www.intel.com/content/www/us/en/io/serial-ata/ahci.html)
- [Avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
