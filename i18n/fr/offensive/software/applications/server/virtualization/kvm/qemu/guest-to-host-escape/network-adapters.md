---
order: 2
title: "Adaptateurs réseau : s'évader de QEMU via les NIC émulées"
description: "QEMU émule plusieurs NIC, la Realtek rtl8139, l'Intel e1000/e1000e et virtio-net, dont les chemins de transmission et de réception analysent des descripteurs et des données de paquet programmés par l'invité. Des failles dans la gestion des descripteurs, dans le traitement de l'offload et du loopback, et dans le réassemblage des paquets donnent un accès hors limites dans le processus QEMU depuis la pile réseau de l'invité."
keywords:
  - e1000
  - rtl8139
  - virtio-net
  - descriptor
  - offload
---

# Adaptateurs réseau

La NIC d'un invité QEMU est émulée, couramment la Realtek rtl8139, l'Intel e1000/e1000e, ou la paravirtuelle virtio-net. Chacune possède des anneaux de descripteurs de transmission et de réception que l'invité programme dans sa mémoire, et QEMU lit ces descripteurs et les buffers de paquet qu'ils référencent pour déplacer le trafic. Le chemin de transmission en particulier analyse des longueurs et des métadonnées d'offload choisies par l'invité, et des fonctionnalités comme le loopback et l'offload de checksum/segmentation ajoutent un traitement côté hôte des données de l'invité, si bien que les modèles de NIC ont produit à plusieurs reprises des bugs de hors-limites et de boucle infinie dans QEMU.

## La surface par modèle

```c
// rtl8139: the transmit path and C+ mode descriptors; offload/loopback handling
//   has produced out-of-bounds reads where a length is trusted during TX processing
// e1000/e1000e: TX descriptor processing, segmentation/checksum offload, and
//   descriptor-ring handling; a TX descriptor length/offload combination used for a
//   copy, or a ring that the model walks without bounding, -> OOB in QEMU
// virtio-net: the virtio-net header and virtqueue descriptors (see virtio devices)
```

```bash
lspci -nn | grep -iE 'ethernet|realtek|intel'   # which NIC model
# an attacker guest NIC driver programs the rings and writes descriptors directly,
# controlling lengths, buffer pointers, and offload fields
```

Les motifs récurrents sont une longueur de transmission ou un champ d'offload que le modèle utilise pour une copie qui dépasse le buffer source, et un traitement d'anneau/descripteur qui boucle ou lit hors limites sur une entrée élaborée.

## Notes d'exploitation

- L'offload côté transmission (segmentation TCP, checksum) est un lieu de bug fréquent car le modèle calcule sur des champs d'en-tête et des longueurs choisis par l'invité ; les modes loopback renvoient des trames élaborées directement à travers l'analyse côté hôte.
- Le modèle e1000 est partagé avec d'autres hyperviseurs (dont VMware), si bien que les bugs e1000 ont souvent une pertinence inter-produits ; virtio-net passe par la couche virtqueue partagée, voir [périphériques virtio](virtio-devices.md).
- L'invité contrôle les anneaux et les descripteurs depuis son pilote, si bien que l'exploitation programme le périphérique directement plutôt que d'envoyer des paquets ordinaires.
- Les primitives atterrissent dans le processus QEMU ; à associer à une fuite et à un grooming, spécifiques à une version de QEMU.

## Références

- [Documentation des périphériques réseau QEMU](https://www.qemu.org/docs/master/system/devices/net.html)
- [Avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM escape : network](https://github.com/WinMin/Awesome-VM-Exploit)
