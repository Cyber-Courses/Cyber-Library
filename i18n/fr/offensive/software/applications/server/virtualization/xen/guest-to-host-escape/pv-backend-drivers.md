---
order: 3
title: "Pilotes backend PV : s'échapper de Xen par les backends de périphériques de dom0"
description: "Les périphériques paravirtuels de Xen sont scindés : un frontend dans l'invité et un backend dans dom0 (ou un domaine pilote) communiquent via un anneau en mémoire partagée. Les backends (blkback, netback et d'autres) analysent les requêtes en anneau publiées par l'invité dans le dom0 privilégié, de sorte qu'une faille dans l'analyse des requêtes ou la gestion des grants donne l'exécution de code dans dom0, qui contrôle chaque domaine."
keywords:
  - pv backend
  - blkback
  - netback
  - ring buffer
  - dom0
---

# Pilotes backend PV

L'I/O paravirtuelle de Xen utilise un modèle split-driver : l'invité exécute un frontend (`blkfront`, `netfront`) qui publie des requêtes dans un anneau en mémoire partagée, et la moitié backend (`blkback`, `netback` et d'autres) tourne dans dom0 ou dans un domaine pilote dédié et les traite. Le backend lit les requêtes en anneau publiées par l'invité, suit les grant references vers les pages de données de l'invité et effectue l'I/O. Parce que cette analyse se produit dans le dom0 privilégié, une faille dans la manière dont un backend valide les requêtes en anneau, le nombre de requêtes, les descripteurs de segments ou les grant references qu'ils transportent, donne l'exécution de code dans dom0, qui administre chaque domaine.

## La surface

```c
// a guest frontend posts requests into the shared ring; the dom0 backend consumes:
//   blkback: block requests with segment descriptors (grant ref + offset + length)
//            per segment; a segment count/length the backend trusts, or a grant ref
//            it maps without proper validation, -> OOB or cross-domain access in dom0
//   netback: packet buffers described by grant refs and lengths; offload/fragment
//            handling with trusted lengths is a classic locus
// ring indices (req_prod/req_cons) are in shared memory; a backend that trusts them
// without bounding against the ring size is another primitive
```

Le format de requête multi-segments du backend bloc et la gestion des fragments et de l'offload du backend réseau sont les foyers récurrents, tous deux analysant des comptes et des longueurs choisis par l'invité dans dom0.

## Notes d'exploitation

- L'exécution de code aboutit dans dom0 (ou dans un domaine pilote), qui est le domaine de contrôle privilégié, de sorte qu'un bug de backend équivaut de fait à une compromission de l'hôte, contrôlant tous les invités, leurs disques et la toolstack.
- Les backends consomment des grant references pour les pages de données, de sorte que ces bugs s'entremêlent souvent avec la surface des [grant tables](grant-tables-and-event-channels.md) ; une mauvaise gestion d'un grant ou d'un anneau est le thème commun.
- Les domaines pilotes (exécutant les backends dans un domaine dédié, moins privilégié) contiennent l'impact à ce domaine plutôt qu'au dom0 complet là où ils sont configurés ; vérifiez si les backends tournent dans dom0 ou dans des domaines pilotes isolés.
- L'attaquant pilote cela depuis un frontend invité qu'il contrôle en publiant des requêtes en anneau forgées ; faites correspondre la version de Xen/du noyau dom0 à l'avis concernant le backend.

## Références

- [Pilotes PV de Xen et modèle split](https://wiki.xenproject.org/wiki/Paravirtualization_(PV))
- [Avis de sécurité Xen (XSA)](https://xenbits.xen.org/xsa/)
