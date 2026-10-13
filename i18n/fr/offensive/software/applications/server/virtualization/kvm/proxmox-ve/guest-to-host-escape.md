---
order: 2
title: "Évasion guest-to-host : sortir d'une machine virtuelle Proxmox"
description: "Proxmox exécute les VM sous KVM avec l'émulation de périphériques QEMU, si bien qu'une évasion guest-to-host de VM cible les mêmes modèles de périphériques QEMU et atterrit dans le processus QEMU par VM sur le nœud Proxmox. Proxmox exécute aussi des conteneurs LXC, dont l'évasion est le breakout de conteneur indépendant du runtime plutôt qu'un bug de modèle de périphériques."
keywords:
  - proxmox escape
  - qemu
  - kvm
  - lxc
  - device model
---

# Évasion guest-to-host

Les VM Proxmox s'exécutent sous KVM avec l'émulation de périphériques QEMU, donc sortir d'une VM Proxmox, c'est le problème QEMU : corrompre le processus QEMU par VM sur le nœud via un modèle de périphériques. Les périphériques et mécanismes accessibles sont ceux de QEMU, documentés sur les pages QEMU, et l'exécution de code atterrit sur le nœud Proxmox. Séparément, Proxmox exécute aussi des conteneurs LXC, et sortir de l'un d'eux n'est pas du tout un bug de modèle de périphériques mais l'évasion de conteneur indépendante du runtime (config sur-privilégiée, montages sensibles, espaces de noms partagés, ou un exploit noyau).

```bash
# VM guest: the QEMU-emulated device inventory (escape surface)
lspci -nn; lsusb; dmesg | grep -i virtio
# LXC container: check the container-escape surface instead
grep CapEff /proc/self/status; cat /proc/self/uid_map; mount | grep -v overlay | head
```

## Évasion de VM : la surface QEMU

Les mécanismes du modèle de périphériques sont partagés avec QEMU :

- gestion des virtqueues virtio, voir [périphériques virtio](../qemu/guest-to-host-escape/virtio-devices.md).
- gestion des NIC émulées, voir [Adaptateurs réseau](../qemu/guest-to-host-escape/network-adapters.md).
- gestion des contrôleurs de stockage, voir [Bloc et SCSI](../qemu/guest-to-host-escape/block-and-scsi.md).
- gestion des contrôleurs USB, voir [Contrôleurs USB](../qemu/guest-to-host-escape/usb-controllers.md).

## Évasion de conteneur LXC

```bash
# Proxmox LXC containers escape via the container primitives, not device models:
# privileged containers, bind mounts of host paths, shared namespaces, kernel bugs
# unprivileged LXC uses a user namespace, which constrains what an escape yields
cat /proc/self/uid_map     # "0 100000 65536" => unprivileged container
```

La voie du conteneur est l'[Évasion de conteneur](../../../containers/container-escape/index.md) indépendante du runtime ; les conteneurs privilégiés Proxmox sont les plus exposés, et les non privilégiés (user-namespacés) limitent le résultat.

## Notes d'exploitation

- Pour les VM, traitez Proxmox comme QEMU : identifiez la version QEMU et faites correspondre un avis de sécurité sur un périphérique, atterrissant dans le processus QEMU par VM sur le nœud.
- Pour LXC, l'évasion est un breakout de conteneur ; un conteneur privilégié Proxmox qui monte aussi des chemins hôtes ou partage des espaces de noms est le cas facile, et les mécanismes sont sur les pages d'évasion de conteneur.
- Dans les deux cas, le point d'atterrissage est le nœud Proxmox ; poursuivez avec [Accès à l'hôte et shell](host-access-and-shell.md) pour atteindre la gestion et le cluster.

## Références

- [Proxmox VE : VM et conteneurs](https://pve.proxmox.com/pve-docs/)
- [Avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
