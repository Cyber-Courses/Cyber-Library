---
order: 3
title: "VMBus : s'évader de Hyper-V via le transport de canal"
description: "VMBus est le transport par ring-buffer qui achemine tout le trafic des périphériques synthétiques entre un invité Hyper-V et la partition racine. L'invité contrôle l'établissement du canal, l'enregistrement mémoire GPADL et les paquets placés dans le ring. Des failles dans la manière dont les endpoints de la partition racine analysent les paquets du ring, les offres de canal et les descripteurs GPADL donnent une corruption mémoire dans la partition racine privilégiée."
keywords:
  - vmbus
  - ring buffer
  - gpadl
  - canal
  - partition racine
---

# VMBus

VMBus est l'épine dorsale des E/S paravirtuelles de Hyper-V : chaque périphérique synthétique communique avec son Virtualization Service Provider dans la partition racine via un canal VMBus, qui est une paire de ring-buffers en mémoire partagée avec l'invité. L'invité participe à la négociation de canal, enregistre de la mémoire auprès de la partition racine par des descripteurs GPADL (Guest Physical Address Descriptor List), et écrit des paquets de requête dans le ring. Tout cela constitue des données contrôlées par l'invité analysées par le code de la partition racine, de sorte que le transport lui-même, avant toute logique de périphérique, est une surface d'évasion : des paquets de ring, des offres de canal ou des descripteurs GPADL malformés peuvent pousser l'endpoint de la partition racine hors limites.

## La surface de transport

```c
// a VMBus packet has a descriptor header (type, offset, length, flags) followed by
// payload; the receiver uses the header's offset/length to locate the payload in
// the ring. A length or offset the root-partition reader trusts, or a ring index
// it advances without bounds checking, yields OOB read/write in the root partition.

// GPADL registers guest memory for a channel: the guest supplies a descriptor list
// of guest physical page ranges. A crafted GPADL (bad ranges, inconsistent counts)
// that the root partition maps/walks without validation is a second primitive.
```

Les index du ring-buffer (pointeurs de lecture et d'écriture) sont en mémoire partagée que l'invité peut influencer, un bug classique est donc un récepteur qui calcule une longueur de paquet ou un index suivant à partir de champs contrôlés par l'invité et lit au-delà du ring ou dans une mémoire non prévue.

## Notes d'exploitation

- VMBus est sous la logique de périphérique, donc un bug au niveau du transport peut affecter plusieurs périphériques synthétiques à la fois ; il est atteint dès qu'un invité peut ouvrir et piloter un canal, ce que fait tout périphérique paravirtuel.
- La gestion GPADL est une surface distincte et de grande valeur : c'est là que la mémoire physique de l'invité est enregistrée auprès de la partition racine, et une mauvaise gestion y fait le pont entre l'adressage contrôlé par l'invité et les mappings de l'hôte.
- Les endpoints de la partition racine s'exécutent avec un privilège élevé (VSP noyau ou processus worker), donc une corruption là est proche du contrôle total de l'hôte ; l'endpoint précis dépend du canal abusé.
- Spécifique au build, comme toujours ; identifiez la version Windows/Hyper-V et faites correspondre à l'avis VMBus ou VSP.

## Références

- [Microsoft: VMBus and synthetic device architecture](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-architecture)
- [MSRC: Hyper-V VMBus research](https://www.microsoft.com/en-us/msrc)
- [Hyper-V internals (community research)](https://github.com/microsoft/MSRC-Security-Research)
