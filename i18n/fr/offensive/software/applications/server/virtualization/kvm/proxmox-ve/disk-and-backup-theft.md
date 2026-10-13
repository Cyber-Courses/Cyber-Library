---
order: 4
title: "Vol de disques et de sauvegardes : récupérer les disques de VM Proxmox et les archives vzdump"
description: "Proxmox stocke les disques de VM sur des backends configurables (qcow2/raw local, LVM, ZFS, Ceph) et les sauvegardes sous forme d'archives vzdump. Avec un accès au nœud, au stockage ou à l'API, un attaquant lit les disques hors ligne avec les outils habituels ou restaure et monte une sauvegarde vzdump, extrayant les données de n'importe quelle VM ou conteneur sans y entrer."
keywords:
  - vzdump
  - qcow2
  - lvm
  - ceph
  - proxmox backup
---

# Vol de disques et de sauvegardes

Proxmox stocke les disques de VM sur le backend pour lequel un stockage est configuré : le stockage en répertoire local contient des fichiers qcow2 ou raw, LVM-thin contient des volumes logiques, et ZFS ou Ceph contiennent leurs propres volumes. Les sauvegardes sont des archives vzdump sur un stockage de sauvegarde ou un Proxmox Backup Server. L'accès à l'un de ces éléments, le système de fichiers du nœud, le backend de stockage ou l'API, permet à un attaquant de lire hors ligne le disque d'une VM ou d'un conteneur, ou de restaurer et monter une sauvegarde, extrayant les données de l'invité sans aucune connexion ni défense interne à l'invité.

## Localiser et lire les disques

```bash
# where disks live depends on the storage type
cat /etc/pve/storage.cfg                   # storage backends and paths
qm config <vmid> | grep -E 'scsi|virtio|ide'   # which volume backs the VM
# local file storage
ls -l /var/lib/vz/images/<vmid>/
guestmount -a /var/lib/vz/images/<vmid>/vm-<vmid>-disk-0.qcow2 -i --ro /mnt/guest
# LVM-thin: activate and mount the logical volume
lvs; lvchange -ay <vg>/vm-<vmid>-disk-0; guestmount -a /dev/<vg>/vm-<vmid>-disk-0 -i --ro /mnt
# ZFS: the zvol under /dev/zvol; Ceph: map the RBD image
```

## Sauvegardes

```bash
# vzdump archives (lzo/zst/gz) on backup storage
ls -l /var/lib/vz/dump/*.vma.* /var/lib/vz/dump/*.tar.*
# restore to a scratch VM/container, or extract a VMA archive to read its disks
qmrestore /var/lib/vz/dump/vzdump-qemu-<vmid>-*.vma.zst 9999   # restore to id 9999
# or extract the VMA to raw disks and mount offline
```

## Notes d'exploitation

- La méthode de lecture dépend du type de stockage : `guestmount`/`qemu-nbd` pour les volumes fichier et bloc, `lvchange -ay` puis montage pour LVM-thin, le périphérique zvol pour ZFS, et `rbd map` pour Ceph ; `storage.cfg` et `qm config` vous indiquent lequel s'applique.
- Les sauvegardes vzdump sont une copie complète et portable d'un invité ; en restaurer une sur un id jetable ou extraire l'archive VMA lit le disque sans toucher à la VM d'origine.
- Le montage hors ligne contourne tous les contrôles de l'invité ; priorisez les magasins d'identifiants (SAM/SYSTEM, `/etc/shadow`, clés) comme pour tout vol de disque.
- Cela s'enchaîne à partir d'un accès au nœud, au stockage ou à l'API ; voir [Accès à l'hôte et shell](host-access-and-shell.md) et le [Plan de gestion](management-plane.md) pour l'obtenir.

## Outils

- [libguestfs / guestmount](https://libguestfs.org/)

## Références

- [Proxmox VE : stockage](https://pve.proxmox.com/pve-docs/chapter-pvesm.html)
- [Proxmox VE : sauvegarde et restauration (vzdump)](https://pve.proxmox.com/pve-docs/chapter-vzdump.html)
