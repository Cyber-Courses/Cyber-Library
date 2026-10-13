---
order: 1
title: "Cartes réseau : s'évader de VirtualBox via les NIC émulées"
description: "VirtualBox émule les cartes Intel e1000, AMD PCnet et virtio-net, dont les pilotes invités programment des anneaux de descripteurs que le processus VM hôte lit pour véhiculer les paquets. Des failles dans la gestion des descripteurs, dans le traitement de l'offload et du loopback, et dans les chemins d'émission PCnet et e1000 donnent un accès hors limites dans le processus hôte, une surface d'évasion VirtualBox récurrente."
keywords:
  - e1000
  - pcnet
  - virtio-net
  - descripteur
  - offload
---

# Cartes réseau

VirtualBox émule plusieurs NIC : l'Intel e1000 (et e1000e), l'AMD PCnet (Am79C97x) et virtio-net. Chacune possède des anneaux de descripteurs d'émission et de réception que l'invité programme dans sa mémoire, et le processus VM hôte lit ces descripteurs et les tampons de paquets pour véhiculer le trafic. Le chemin d'émission et la gestion de l'offload analysent des longueurs et des métadonnées choisies par l'invité, donc les modèles de NIC ont produit à plusieurs reprises des lectures et écritures hors limites dans VirtualBox, la gestion des descripteurs d'émission PCnet et e1000 étant des foyers notables.

## La surface

```c
// e1000: TX descriptor processing and segmentation/checksum offload; a TX length or
//   offload combination the model copies without bounding -> OOB read into the host
// PCnet: the legacy Am79C97x transmit/receive descriptor rings; crafted descriptors
//   (lengths, chaining) have driven out-of-bounds access in the emulation
// virtio-net: the virtio-net header and virtqueue descriptors (shared virtio surface)
```

```bash
lspci -nn | grep -iE 'ethernet|intel|amd'   # which NIC model is configured
# an attacker guest NIC driver programs the rings and descriptors directly
```

## Notes d'exploitation

- Le traitement côté émission et l'offload sont les foyers de bugs habituels : l'hôte calcule sur des longueurs et des champs d'en-tête choisis par l'invité, et les modes loopback réinjectent des trames forgées dans l'analyse de l'hôte.
- Le modèle PCnet est un périphérique hérité toujours proposé et possède son propre historique de gestion des descripteurs ; le modèle e1000 est partagé dans l'esprit avec d'autres hyperviseurs, ce qui lui donne une pertinence inter-produits.
- L'invité contrôle les anneaux et les descripteurs depuis son pilote, donc l'exploitation programme directement le périphérique plutôt que d'envoyer des paquets normaux ; virtio-net passe par la couche virtqueue partagée.
- Les primitives atterrissent dans le processus VM ; spécifiques à la version, à combiner avec une fuite.

## Références

- [VirtualBox manual: networking](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox network research](https://www.zerodayinitiative.com/blog)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
