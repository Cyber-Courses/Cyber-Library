---
order: 1
title: "Commutateur virtuel : s'évader de Hyper-V via le datapath réseau"
description: "Le commutateur virtuel Hyper-V transmet le trafic réseau des VM dans la partition racine, avec une pile de filtres extensible et les chemins de carte réseau synthétique et émulée qui l'alimentent. Les paquets contrôlés par l'invité et les requêtes du VSP réseau synthétique y sont analysés, de sorte que des failles dans le commutateur, ses extensions ou le VSP réseau donnent une corruption mémoire dans la pile réseau de la partition racine."
keywords:
  - commutateur virtuel hyper-v
  - vmswitch
  - netvsp
  - analyse de paquets
  - partition racine
---

# Commutateur virtuel

Le commutateur virtuel Hyper-V (`vmswitch`) transmet le trafic réseau de chaque VM, s'exécutant dans le noyau de la partition racine. Le trafic de l'invité l'atteint via le VSP réseau synthétique (`netvsp`) par VMBus ou via une carte réseau émulée, et le commutateur applique une pile de filtres extensible avant de transmettre. Parce que le commutateur et le VSP réseau analysent des paquets et des structures de requête contrôlés par l'invité dans la partition racine privilégiée, des failles là, dans l'analyse de paquets, la gestion d'offload ou le traitement des rings et descripteurs du VSP, corrompent la pile réseau de la partition racine et ont constitué une surface d'évasion Hyper-V notable.

## La surface

```c
// netvsp receives send/receive buffer descriptors and RNDIS control/data messages
// from the guest over VMBus. Primitives:
//  - an RNDIS message with a length or offset field vmswitch/netvsp trusts
//  - send/receive buffer section descriptors with out-of-range offsets or counts
//  - offload (checksum/segmentation) metadata the switch parses from guest packets
// a trusted length used for a copy, or an index not bounded, -> OOB in the root partition
```

RNDIS est le protocole de contrôle qu'utilise la carte réseau synthétique ; ses messages (set/query OIDs, descripteurs de paquets) sont analysés côté hôte, et un RNDIS malformé a été une source récurrente de bugs `vmswitch`.

## Notes d'exploitation

- Le chemin de la carte réseau synthétique (netvsp plus vmswitch) est une cible plus riche que la carte réseau émulée, car il analyse le protocole RNDIS et le modèle de descripteurs de buffer d'émission/réception dans le noyau de la partition racine.
- `vmswitch` s'exécute dans le noyau de la partition racine, donc la corruption est une primitive de niveau noyau sur l'hôte, parmi les évasions Hyper-V au plus fort impact.
- L'invité pilote cela en envoyant des messages RNDIS et des descripteurs de buffer forgés depuis son pilote de carte réseau ; un pilote invité attaquant les contrôle directement.
- Le transport est [VMBus](vmbus.md) ; le VSP réseau est l'un des [périphériques synthétiques](synthetic-devices.md), et cette page traite les spécificités du commutateur et de RNDIS.

## Références

- [Microsoft: Hyper-V virtual switch](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v-virtual-switch/hyper-v-virtual-switch)
- [MSRC: vmswitch research](https://www.microsoft.com/en-us/msrc)
