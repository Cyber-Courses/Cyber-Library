---
order: 1
title: "Hypercalls : s'échapper de Xen par l'interface invité-vers-hyperviseur"
description: "Les hypercalls sont les appels directs de l'invité Xen vers l'hyperviseur, couvrant la gestion mémoire, les mises à jour de tables de pages, les I/O et plus encore. L'hyperviseur les exécute dans son propre contexte avec des arguments fournis par l'invité, de sorte qu'une faille dans la validation des arguments d'un hypercall ou dans les sous-opérations (memory, grant, mmu, physdev) corrompt ou subvertit l'hyperviseur lui-même, la cible d'évasion la plus privilégiée."
keywords:
  - hypercall
  - hyperviseur xen
  - mmu update
  - memory op
  - validation d'arguments
---

# Hypercalls

Les hypercalls sont l'équivalent des appels système pour Xen : un invité déclenche un trap vers l'hyperviseur pour demander des opérations privilégiées, la gestion mémoire (`mmu_update`, `mmuext_op`), l'allocation mémoire (`memory_op`), les opérations de grant table (`grant_table_op`), l'accès aux périphériques physiques (`physdev_op`), les event channels, et bien d'autres. L'hyperviseur les exécute dans son propre contexte hautement privilégié en utilisant des arguments fournis par l'invité, souvent des pointeurs vers la mémoire de l'invité qu'il doit copier et valider. Une faille dans cette validation, une borne manquante, une confusion de types entre sous-opérations, une mise à jour de table de pages qui échappe à la plage autorisée de l'invité, subvertit directement l'hyperviseur, ce qui constitue l'évasion la plus grave car il se situe en dessous de dom0 et de tous les autres domaines.

## La surface

```c
// a guest issues a hypercall with a sub-command and guest-memory arguments:
//   HYPERVISOR_mmu_update(reqs, count, ...)       // batched page-table updates
//   HYPERVISOR_memory_op(cmd, &arg)               // allocate/map/exchange frames
//   HYPERVISOR_grant_table_op(cmd, uop, count)    // grant map/copy/transfer
//   HYPERVISOR_mmuext_op(...), HYPERVISOR_physdev_op(...)
// the hypervisor copies and validates these from guest memory. Primitives:
//  - an MMU update that maps a frame the guest should not own (page ownership checks)
//  - a count/index in a batched op the hypervisor trusts -> OOB over the request array
//  - type confusion between sub-op structures sharing a union
```

Les hypercalls de table de pages et de gestion mémoire sont les plus riches car ils manipulent directement les mappings que l'hyperviseur applique ; un bug qui permet à un invité de mapper une page qu'il ne possède pas est une primitive de lecture/écriture vers les autres domaines ou l'hyperviseur.

## Notes d'exploitation

- Un succès aboutit dans l'hyperviseur, en dessous de dom0, c'est donc l'évasion Xen au plus fort impact : elle peut atteindre la mémoire de n'importe quel domaine et l'état propre de l'hyperviseur.
- Les invités PV disposent de l'accès aux hypercalls le plus large (ils utilisent directement les hypercalls pour la gestion des tables de pages), de sorte que les configurations PV classiques exposent la plus grande surface ; PVH/HVM la réduisent sans la supprimer.
- L'attaquant émet des hypercalls forgés depuis un noyau d'invité qu'il contrôle ; les opérations groupées (tableaux de requêtes) sont un foyer récurrent de bugs de count/index.
- Les avis de sécurité Xen (XSA) cataloguent les bugs d'hypercall par sous-opération ; identifiez la version de Xen et faites correspondre.

## Références

- [Documentation des hypercalls Xen](https://xenbits.xen.org/docs/)
- [Avis de sécurité Xen (XSA)](https://xenbits.xen.org/xsa/)
- [Google Project Zero : recherche sur les hypercalls Xen](https://googleprojectzero.blogspot.com/)
