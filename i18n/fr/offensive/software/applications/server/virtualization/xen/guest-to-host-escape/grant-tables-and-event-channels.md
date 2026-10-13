---
order: 2
title: "Grant tables et event channels : s'échapper de Xen par la machinerie inter-domaines"
description: "Les grant tables permettent à un domaine Xen de partager ses pages mémoire avec un autre, et les event channels délivrent des signaux inter-domaines. Les pilotes paravirtuels s'appuient sur les deux. Des failles dans la gestion des opérations map/copy/transfer des grants, dans le comptage de références lors du démantèlement d'un grant, et dans l'allocation d'event channels donnent un accès mémoire inter-domaines ou une corruption de l'hyperviseur atteignable depuis un invité."
keywords:
  - grant tables
  - event channels
  - grant reference
  - mémoire partagée
  - inter-domaines
---

# Grant tables et event channels

Le modèle split-driver de Xen exige que les domaines partagent de la mémoire et se signalent mutuellement, et deux mécanismes le permettent. Les grant tables permettent à un domaine d'accorder à un autre domaine un accès (map ou copy) à des pages spécifiques, identifiées par des grant references ; c'est ainsi qu'un invité expose ses buffers en anneau (ring buffers) d'I/O et ses pages de données au backend dom0. Les event channels sont le signal inter-domaines léger (une notification de type interruption). Les deux sont gérés via des hypercalls et suivis par l'hyperviseur, de sorte que des failles dans la gestion des opérations map/copy/transfer des grants, dans le comptage de références qui détermine quand une page accordée peut être réclamée, et dans l'allocation et le binding d'event channels, livrent un accès mémoire inter-domaines ou une corruption de l'état de l'hyperviseur depuis un invité.

## La surface

```c
// grant operations (via HYPERVISOR_grant_table_op):
//   GNTTABOP_map_grant_ref   - map a page another domain granted
//   GNTTABOP_copy            - copy to/from a granted page
//   GNTTABOP_transfer        - transfer page ownership
// Primitives:
//  - a grant/copy with a length or offset the handler trusts -> OOB access
//  - reference-count errors on map/unmap letting a page be reused while still mapped
//    (a use-after-free across domains, or a guest retaining access after teardown)
//  - event-channel allocation/binding flaws (exhaustion, confusion of port types)
```

L'angle du comptage de références est historiquement important : si un invité peut maintenir un grant mappé après que le propriétaire croit avoir réclamé la page, il conserve un accès à de la mémoire que l'hyperviseur a réassignée, ouvrant un pont vers dom0 ou un autre domaine.

## Notes d'exploitation

- La gestion de la longueur et de l'offset des opérations grant copy/map est la surface OOB directe ; le cycle de vie du comptage de références (map, use, unmap, reclaim) est le plus subtil et récurremment bogué, produisant des use-after-free inter-domaines.
- Ces mécanismes sous-tendent les backends PV, ils sont donc atteignables par tout invité utilisant du bloc ou du réseau paravirtuel ; l'attaquant les pilote depuis un frontend/noyau d'invité qu'il contrôle.
- L'impact va de la lecture/écriture mémoire inter-domaines (atteignant dom0 ou les pairs) à la corruption de l'hyperviseur, selon le bug ; les deux sont graves.
- Catalogués dans les XSA ; faites correspondre la version de Xen. Les backends qui utilisent ces mécanismes sont couverts dans [Pilotes backend PV](pv-backend-drivers.md).

## Références

- [Documentation des grant tables Xen](https://xenbits.xen.org/docs/)
- [Avis de sécurité Xen (XSA)](https://xenbits.xen.org/xsa/)
