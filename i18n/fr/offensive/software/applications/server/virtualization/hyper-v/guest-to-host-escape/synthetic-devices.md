---
order: 2
title: "Périphériques synthétiques : s'évader de Hyper-V via les périphériques adossés aux VSP"
description: "Les périphériques synthétiques de Hyper-V (stockage, réseau, vidéo, HID, et d'autres) sont des périphériques paravirtuels dont les pilotes côté invité dialoguent avec des Virtualization Service Providers dans la partition racine via VMBus. Les VSP analysent les requêtes et descripteurs fournis par l'invité, de sorte qu'une faille de sûreté mémoire dans un VSP, notamment les providers vidéo et stockage synthétiques, donne l'exécution de code dans la partition racine privilégiée."
keywords:
  - périphériques synthétiques
  - vsp
  - storvsp
  - synthvid
  - partition racine
---

# Périphériques synthétiques

Les E/S performantes de Hyper-V utilisent des périphériques synthétiques : l'invité exécute un pilote Virtualization Service Consumer (VSC) qui communique via VMBus avec un Virtualization Service Provider (VSP) dans la partition racine. Il existe des providers synthétiques de stockage, réseau, vidéo, clavier/souris (HID) et d'autres. Chaque VSP analyse les requêtes et descripteurs que son VSC invité envoie, chacun est donc une surface d'évasion, et les providers vidéo (`synthvid`) et stockage (`storvsp`) synthétiques ont historiquement été productifs en raison de la complexité de ce qu'ils analysent.

## La surface par périphérique

```c
// a synthetic device request carries a device-specific message over VMBus; the VSP
// in the root partition parses it. Typical primitives:
//  - synthetic video: a message describing a framebuffer/mode with dimensions or a
//    length the VSP trusts, or a situation update referencing a region out of range
//  - synthetic storage (storvsp): a SCSI request packet whose data-transfer length
//    or scatter-gather description the provider uses without bounds checking
//  - synthetic HID/keyboard: report descriptors and input reports parsed in the host
```

L'invité contrôle entièrement ces messages, donc les classes de bugs sont celles habituelles : une longueur ou un compteur auquel le VSP fait confiance, un index qu'il ne borne pas, ou un use-after-free sur un objet de requête au cours du cycle de vie du canal, le tout atterrissant dans la mémoire de la partition racine.

## Notes d'exploitation

- Le VSP vidéo synthétique est une cible récurrente car il analyse des descripteurs de mode et de région avec une géométrie contrôlée par l'invité ; le VSP stockage synthétique analyse des requêtes de niveau SCSI avec des descriptions de transfert contrôlées par l'invité.
- Les VSP s'exécutent dans la partition racine (plusieurs en tant que composants noyau), donc la corruption est de haut privilège ; le VSP que vous atteignez détermine si vous atterrissez dans un VSP noyau ou dans le processus worker.
- L'invité atteint un VSP donné en pilotant son VSC via VMBus ; depuis un pilote invité contrôlé par l'attaquant vous émettez directement des messages de périphérique forgés plutôt que par la pile normale.
- Ceux-ci sont spécifiques à la version ; faites correspondre le build Hyper-V à l'avis du VSP, et notez que le transport sous-jacent, [VMBus](vmbus.md), est une surface distincte.

## Références

- [Microsoft: Hyper-V synthetic devices and VSP/VSC](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V VSP research](https://www.microsoft.com/en-us/msrc)
