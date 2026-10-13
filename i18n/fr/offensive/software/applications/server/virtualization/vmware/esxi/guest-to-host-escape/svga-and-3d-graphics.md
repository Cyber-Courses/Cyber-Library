---
order: 1
title: "SVGA et graphismes 3D : s'évader d'ESXi via le périphérique graphique"
description: "Le périphérique VMware SVGA II expose une FIFO de commandes et une interface de rendu 3D que le pilote invité remplit de commandes et de définitions de surfaces. Le processus vmx analyse ce flux contrôlé par l'invité, y compris les opérations de shaders et de surfaces, de sorte qu'une faille de sûreté mémoire dans la gestion des commandes 3D donne une écriture hors limites dans le processus hôte, une évasion guest-to-host récurrente."
keywords:
  - svga
  - vmwgfx
  - fifo
  - accélération 3d
  - gmr
---

# SVGA et graphismes 3D

Le périphérique VMware SVGA II est l'adaptateur d'affichage de l'invité, piloté sous Linux par `vmwgfx`. C'est bien plus qu'un framebuffer : l'invité soumet un flux de commandes via une FIFO et, avec l'accélération 3D activée, un riche ensemble d'opérations de rendu (surfaces, contextes, shaders, draw calls) que le processus `vmx` interprète. Parce que ce flux de commandes et les objets qu'il référence sont entièrement contrôlés par l'invité et analysés dans le processus hôte, le chemin 3D est l'une des surfaces d'évasion ESXi les plus productives, avec des bugs répétés de hors-limites et de type-confusion dans la gestion des surfaces et des shaders.

## La FIFO de commandes

L'invité active la FIFO via les registres du périphérique et écrit des mots de commande dans la région mémoire de la FIFO ; `vmx` les consomme :

```c
// SVGA registers are accessed via the index/value I/O ports (SVGA_INDEX/_VALUE)
// enable the device and FIFO, then submit commands into the FIFO memory
svga_write(SVGA_REG_ENABLE, 1);
svga_write(SVGA_REG_CONFIG_DONE, 1);         // FIFO active
// each command begins with an SVGA_CMD_* or SVGA_3D_CMD_* id followed by a body
// 2D example: SVGA_CMD_UPDATE; 3D example: SVGA_3D_CMD_SURFACE_DEFINE, _SHADER_DEFINE,
// _DRAW_PRIMITIVES, each with guest-chosen sizes and object references
```

## Le modèle d'objets 3D

Le rendu 3D définit des surfaces (textures et render targets), des contextes et des shaders, et émet des draw calls qui les référencent. Le côté hôte alloue et suit ces objets depuis des descripteurs fournis par l'invité, donc les classes de bugs sont : une surface ou un mip-level défini avec des dimensions qui débordent une allocation, un shader avec une longueur à laquelle l'analyseur fait confiance, un draw call référençant un id d'objet que l'hôte déréférence sans validation (type-confusion), et des régions mémoire invité (GMR) utilisées pour le DMA avec des offsets hors plage.

```c
// a crafted SURFACE_DEFINE with inconsistent face/mip sizes, or a SHADER_DEFINE
// with an oversized bytecode length, drives the host-side allocation/parse out of
// bounds; DRAW/PRESENT referencing a stale or wrong-typed object id triggers UAF
```

## Notes d'exploitation

- La 3D doit être accessible pour la surface la plus riche ; le périphérique et la FIFO sont activés via les registres SVGA depuis l'invité, et le pilote `vmwgfx` démontre la séquence exacte de mise en place des registres et de la FIFO.
- Les primitives courantes sont une écriture hors limites depuis une taille de surface/shader à laquelle l'hôte fait confiance, et une type-confusion depuis un id d'objet référencé entre types de commandes ; les deux atterrissent dans le tas de `vmx`.
- Le DMA basé sur GMR permet à l'invité de pointer le périphérique vers la mémoire invitée avec des offsets choisis par l'attaquant, ce qui se combine aux bugs d'analyse pour construire une lecture/écriture contrôlée.
- Une évasion fonctionnelle associe une fuite (readback de surface ou commande de fuite d'information) pour défaire l'ASLR de `vmx` à la primitive d'écriture ; elles sont spécifiques au build, donc faites correspondre la version d'ESXi à un bug 3D connu.

## Références

- [VMware SVGA device interface (open-vm-tools / vmwgfx headers)](https://github.com/vmware/open-vm-tools)
- [Zero Day Initiative: VMware SVGA 3D research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
