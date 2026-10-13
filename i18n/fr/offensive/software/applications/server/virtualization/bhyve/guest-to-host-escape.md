---
order: 2
title: "Évasion guest-to-host : sortir d'une machine virtuelle bhyve"
description: "Un invité bhyve s'évade en corrompant le processus bhyve en espace utilisateur qui émule ses périphériques. La surface accessible comprend les modèles de périphériques de bhyve, la famille virtio, le contrôleur de stockage AHCI, la carte réseau e1000, les contrôleurs USB et le framebuffer, chacun analysant les écritures de registres et les descripteurs DMA pilotés par l'invité, une faille donnant l'exécution de code dans le processus bhyve sur l'hôte FreeBSD."
keywords:
  - évasion bhyve
  - virtio
  - ahci
  - e1000
  - modèle de périphérique
---

# Évasion guest-to-host

bhyve émule les périphériques d'une VM dans un processus en espace utilisateur propre à chaque VM, le module noyau `vmm.ko` accélérant le CPU. S'évader d'un invité bhyve revient à faire manipuler incorrectement par ce processus `bhyve` des données contrôlées par l'invité dans un modèle de périphérique. La surface est l'ensemble des périphériques de bhyve : les périphériques virtio (virtio-blk, virtio-net, virtio-9p, virtio-console), le contrôleur SATA AHCI, la carte réseau e1000, le contrôleur USB (XHCI) et le framebuffer. Chacun lit les écritures de registres et les descripteurs DMA de l'invité, de sorte qu'une faille de sûreté mémoire donne l'exécution de code dans le processus `bhyve` sur l'hôte FreeBSD.

```bash
# the emulated devices visible in the guest (escape surface)
pciconf -lv 2>/dev/null | grep -iE 'virtio|ahci|e1000|xhci'   # (FreeBSD guest)
lspci -nn 2>/dev/null | grep -iE 'virtio|ahci|intel|usb'      # (Linux guest)
```

## La surface des périphériques

```c
// bhyve device models parse guest-controlled structures:
//  - virtio (blk/net/9p/console): virtqueue descriptors (addr/len/flags), indirect
//    descriptors, and per-device headers; a trusted length/index -> OOB in bhyve
//  - AHCI: command list + PRD tables with guest addresses/counts for DMA
//  - e1000: TX/RX descriptors and offload fields
//  - XHCI: command/transfer rings of TRBs with link/length fields
// the mechanisms match the equivalent QEMU device models
```

Les classes de bugs sont celles classiques de l'émulation de périphériques : une longueur ou un compteur auquel le modèle fait confiance, un index qu'il ne borne pas, un use-after-free sur un objet de requête, et parce que bhyve met l'accent sur virtio, la gestion des virtqueues (mécanisme partagé, voir [périphériques virtio QEMU](../kvm/qemu/guest-to-host-escape/virtio-devices.md)) est une surface primaire.

## Notes d'exploitation

- L'exécution de code atterrit dans le processus `bhyve` en espace utilisateur sur l'hôte FreeBSD ; il s'exécute avec les privilèges de celui qui a lancé la VM, donc une élévation de privilèges locale supplémentaire peut être nécessaire pour un contrôle complet de l'hôte, comme avec d'autres configurations de type 2.
- La conception épurée et centrée sur virtio de bhyve fait des chemins virtio et AHCI les surfaces primaires ; les mécanismes reflètent ceux de QEMU, donc cette analyse de périphériques se transpose.
- L'invité pilote les modèles de périphériques depuis un pilote contrôlé, en postant directement des descripteurs forgés ; les primitives atterrissent dans le tas de bhyve et sont spécifiques à la version.
- Les avis de sécurité FreeBSD suivent les bugs de périphériques bhyve ; identifiez la version de FreeBSD/bhyve.

## Références

- [bhyve(8) and device models](https://man.freebsd.org/cgi/man.cgi?query=bhyve)
- [FreeBSD security advisories (bhyve)](https://www.freebsd.org/security/advisories/)
