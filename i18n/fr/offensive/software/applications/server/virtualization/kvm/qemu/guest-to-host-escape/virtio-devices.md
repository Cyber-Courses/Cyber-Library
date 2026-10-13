---
order: 1
title: "périphériques virtio : s'évader de QEMU via la famille virtqueue"
description: "virtio est le framework de périphériques paravirtuels de QEMU : le pilote invité et QEMU partagent des virtqueues, des structures en anneau en mémoire invité contenant des descripteurs qui pointent vers des buffers invités. QEMU parcourt ces chaînes de descripteurs pour effectuer les E/S, si bien que des failles dans la gestion des descripteurs, la longueur de chaîne, les descripteurs indirects ou la configuration virtio, donnent un accès hors limites dans le processus QEMU."
keywords:
  - virtio
  - virtqueue
  - vring
  - descriptor
  - indirect descriptor
---

# périphériques virtio

virtio est le framework d'E/S paravirtuel partagé par virtio-net, virtio-blk, virtio-scsi, virtio-gpu et d'autres. Le pilote invité et QEMU communiquent via des virtqueues : chacune est un vring avec une table de descripteurs, un anneau disponible et un anneau utilisé, le tout en mémoire invité. Un descripteur contient une adresse physique invité, une longueur et des flags ; les descripteurs s'enchaînent, et un descripteur indirect pointe vers une table supplémentaire de descripteurs. QEMU lit ces structures contrôlées par l'invité pour localiser les buffers de chaque opération, si bien que la gestion des chaînes de descripteurs, des longueurs, des tables indirectes et de l'espace de configuration du périphérique est la surface d'évasion, et elle est large car chaque périphérique virtio la partage.

## La surface virtqueue

```c
// a vring descriptor (guest-controlled):
struct vring_desc { uint64_t addr; uint32_t len; uint16_t flags; uint16_t next; };
// QEMU maps addr for len bytes and follows next when VRING_DESC_F_NEXT is set,
// or treats the buffer as a descriptor table when VRING_DESC_F_INDIRECT is set.
// Primitives a malicious guest driver builds:
//  - a descriptor length QEMU uses for a copy that exceeds the mapped region
//  - a chain or indirect table forming a loop or huge count -> over-read/-write
//  - an available/used ring index advanced past the queue size
//  - per-device config/header fields (e.g. virtio-net header, virtio-gpu commands)
//    with lengths or counts QEMU trusts
```

Parce que l'invité écrit directement la table de descripteurs et les index d'anneau, il contrôle les adresses, les longueurs et les liens que QEMU suit ; les bugs récurrents sont une longueur de confiance utilisée pour une opération mémoire et un index ou un compte non borné par rapport à la queue ou au buffer.

## Spécificités par périphérique

```bash
# virtio-net: the virtio-net header and packet buffers; offload fields
# virtio-blk/scsi: request headers with sector/length fields
# virtio-gpu: a command stream (resource create, transfer, present) with guest sizes
lspci -nn | grep -i virtio; dmesg | grep -i virtio
```

virtio-gpu se distingue par l'analyse d'un flux de commandes avec des descripteurs de ressource et de transfert, similaire dans l'esprit à la surface 3D SVGA de VMware ; virtio-net et virtio-blk exposent l'analyse d'en-têtes et de requêtes.

## Notes d'exploitation

- La couche virtqueue partagée signifie qu'un bug de gestion de descripteur peut être atteint via n'importe quel périphérique virtio présent ; les descripteurs indirects sont un point chaud classique car ils ajoutent une seconde table contrôlée par l'invité que QEMU doit borner.
- virtio-gpu élargit la surface avec un modèle de commandes/ressources par-dessus la virtqueue, une cible d'analyse plus riche proche des chemins 3D d'autres hyperviseurs.
- L'invité pilote cela depuis un pilote virtio contrôlé qui écrit des descripteurs et des anneaux bruts, et non via la pile noyau normale, offrant un contrôle précis des structures que QEMU lit.
- Les primitives atterrissent dans le tas du processus QEMU ; une évasion complète ajoute une fuite et un grooming du tas, et est spécifique à une version de QEMU.

## Références

- [Spécification virtio (OASIS)](https://docs.oasis-open.org/virtio/virtio/v1.2/virtio-v1.2.html)
- [Avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM escape : virtio](https://github.com/WinMin/Awesome-VM-Exploit)
