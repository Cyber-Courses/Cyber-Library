---
order: 4
title: "Vol de disques et de snapshots : récupérer les images qcow2 et l'état sauvegardé"
description: "Les invités KVM utilisent des images disque qcow2 (ou raw) et stockent les snapshots à l'intérieur du qcow2 ou sous forme d'overlays externes, l'état de VM sauvegardé contenant la mémoire. L'accès aux fichiers image permet à un attaquant de lire le système de fichiers de l'invité hors ligne avec libguestfs ou qemu-nbd, extrayant identifiants et données sans entrer dans la VM, et de lire la mémoire de l'état sauvegardé pour les secrets capturés au moment du snapshot."
keywords:
  - qcow2
  - qemu-nbd
  - libguestfs
  - snapshot
  - offline access
---

# Vol de disques et de snapshots

Les machines virtuelles KVM stockent leurs disques sous forme de fichiers image qcow2 ou raw, les snapshots internes conservés à l'intérieur du qcow2 ou sous forme de fichiers overlay externes, et l'état de VM sauvegardé (y compris la mémoire) écrit lors d'une sauvegarde gérée ou d'un snapshot. L'accès à ces fichiers, sur l'hôte, un magasin d'images ou une sauvegarde, permet à un attaquant de lire le système de fichiers de l'invité hors ligne et d'en extraire tout le contenu sans aucune connexion ni défense interne à l'invité. Les fichiers d'état sauvegardé capturent en plus la mémoire de l'invité, si bien que les secrets présents au moment de la sauvegarde sont récupérables.

## Localiser et monter les images

```bash
# typical locations
ls -l /var/lib/libvirt/images/ /var/lib/vz/images/ 2>/dev/null
virsh domblklist <dom>                         # which image backs a domain
# read a qcow2 offline with libguestfs (handles qcow2 and snapshots natively)
guestmount -a disk.qcow2 -i --ro /mnt/guest
cat /mnt/guest/etc/shadow /mnt/guest/root/.ssh/id_* 2>/dev/null
# or expose it as a block device with qemu-nbd and mount a partition
modprobe nbd; qemu-nbd --connect=/dev/nbd0 --read-only disk.qcow2
fdisk -l /dev/nbd0; mount -o ro /dev/nbd0p1 /mnt/guest
```

## Snapshots et état sauvegardé

```bash
# internal snapshots inside a qcow2
qemu-img snapshot -l disk.qcow2
# external snapshot overlays chain to a backing file; mount the active overlay
qemu-img info --backing-chain disk-overlay.qcow2
# saved VM state (virsh managedsave / snapshot with memory) holds guest RAM
ls -l /var/lib/libvirt/qemu/save/*.save 2>/dev/null   # carve for in-memory secrets
```

## Notes d'exploitation

- Le montage hors ligne avec `guestmount` ou `qemu-nbd` contourne tous les contrôles côté invité ; priorisez les ruches SAM/SYSTEM de Windows ou le `/etc/shadow` de Linux et les clés SSH/cloud qui déverrouillent d'autres systèmes.
- Les overlays de snapshot externes nécessitent la chaîne de backing pour lire l'état courant ; `qemu-img info --backing-chain` montre la chaîne complète à assembler.
- Les fichiers d'état sauvegardé sont une image mémoire au moment de la sauvegarde et contiennent des secrets déchiffrés ; extrayez-les avec un outil d'analyse forensique mémoire.
- L'accès en écriture aux images permet une altération avant démarrage : injecter une charge de démarrage dans le système de fichiers d'une VM éteinte. L'accès au fichier seul équivaut au vol complet des données de l'invité pour chaque VM du magasin.

## Outils

- [libguestfs / guestmount](https://libguestfs.org/)
- [qemu-img et qemu-nbd](https://www.qemu.org/docs/master/tools/)

## Références

- [QEMU : qcow2 et images disque](https://www.qemu.org/docs/master/system/images.html)
- [libvirt : gestion des snapshots](https://libvirt.org/formatsnapshot.html)
