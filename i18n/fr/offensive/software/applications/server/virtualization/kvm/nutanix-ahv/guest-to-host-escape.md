---
order: 2
title: "Évasion guest-to-host : sortir d'une machine virtuelle Nutanix AHV"
description: "AHV exécute les VM sous KVM avec une émulation de périphériques dérivée de QEMU, donc une évasion guest-to-host cible les mêmes modèles de périphériques QEMU, virtio, cartes réseau, stockage, USB, et atterrit dans le processus QEMU par VM sur l'hôte AHV. De là, l'attaquant atteint le nœud et, via la relation avec la Controller VM, la fabrique de stockage et le plan de gestion."
keywords:
  - évasion nutanix ahv
  - qemu
  - kvm
  - modèle de périphérique
  - controller vm
---

# Évasion guest-to-host

Parce qu'AHV utilise KVM avec une émulation de périphériques dérivée de QEMU, s'évader d'un invité AHV revient au problème QEMU : corrompre le processus QEMU par VM sur le nœud AHV via un modèle de périphérique. La surface accessible et les mécanismes sont ceux de QEMU, virtqueues virtio, cartes réseau émulées, contrôleurs de stockage, USB, donc le détail par périphérique figure sur les pages QEMU. Ce qui diffère, c'est l'environnement d'atterrissage et le pivot ultérieur : l'exécution de code atterrit sur un nœud AHV dont le stockage et la gestion transitent par une Controller VM, de sorte que l'évasion est une étape vers le nœud, la CVM et le cluster.

```bash
# from the guest: the QEMU-emulated device inventory (escape surface)
lspci -nn; lsusb; dmesg | grep -i virtio
```

Les mécanismes des modèles de périphériques sont partagés avec QEMU :

- gestion des virtqueues virtio et des descripteurs indirects, voir [périphériques virtio](../qemu/guest-to-host-escape/virtio-devices.md).
- gestion des descripteurs et de l'offload des cartes réseau émulées, voir [Cartes réseau](../qemu/guest-to-host-escape/network-adapters.md).
- gestion des commandes et des PRD des contrôleurs de stockage, voir [Block et SCSI](../qemu/guest-to-host-escape/block-and-scsi.md).
- gestion des rings des contrôleurs USB, voir [Contrôleurs USB](../qemu/guest-to-host-escape/usb-controllers.md).

## Ce qui diffère sur AHV

```bash
# after landing in the QEMU process on the AHV node:
#  - the node's storage I/O is served by the local Controller VM; reaching the CVM
#    (over its internal network, typically 192.168.5.0/24) is the pivot to the fabric
#  - node and CVM credentials/keys enable moving to Prism and other nodes
ip route; ip -4 addr   # the internal CVM network is reachable from the host
```

## Notes d'exploitation

- La primitive d'évasion et la mécanique par périphérique sont celles de QEMU ; identifiez la version d'AHV (et donc de QEMU) et faites correspondre à l'avis du périphérique, exactement comme pour les [exploits d'évasion connus QEMU](../qemu/known-escape-exploits.md).
- La valeur spécifique à AHV est le chemin ultérieur : depuis le nœud, la Controller VM locale médie tout le stockage et exécute les services de gestion, donc un point d'appui sur le nœud est une étape vers tout le cluster via le réseau interne de la CVM.
- Nutanix ajoute son propre confinement et ses propres services autour de QEMU ; évaluez le sandboxing du nœud comme pour n'importe quel hôte KVM, voir [Accès à l'hôte et shell](host-access-and-shell.md).

## Références

- [Nutanix AHV architecture](https://www.nutanix.com/products/ahv)
- [QEMU security advisories](https://www.qemu.org/docs/master/system/security.html)
- [Nutanix security advisories](https://www.nutanix.com/support-services/security-advisories)
