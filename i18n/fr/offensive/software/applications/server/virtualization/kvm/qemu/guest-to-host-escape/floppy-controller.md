---
order: 5
title: "Contrôleur de disquette : s'évader de QEMU via le FDC legacy"
description: "Le contrôleur de disquette émulé par QEMU traite les commandes de l'invité à travers une FIFO interne de taille fixe. Un invité qui émet certaines commandes de lecture/écriture peut pousser l'index de la FIFO au-delà du buffer, le bug de la classe VENOM, donnant une lecture et une écriture hors limites dans le processus QEMU. Le contrôleur est présent même lorsqu'aucun lecteur de disquette n'est attaché."
keywords:
  - floppy controller
  - fdc
  - venom
  - fifo
  - qemu escape
---

# Contrôleur de disquette

Le contrôleur de disquette (FDC) de QEMU est un modèle de périphérique petit et ancien qui a pourtant produit l'une des évasions les plus connues, VENOM. Le contrôleur traite les commandes de l'invité à travers une FIFO de registre de données interne de taille fixe. Pour certaines commandes, le gestionnaire avance un index dans cette FIFO à mesure qu'il consomme des paramètres ou transfère des données, et une faille dans le bornage de cet index laisse un invité le pousser au-delà de la fin du tableau, lisant et écrivant la mémoire adjacente du processus QEMU. Le contrôleur est instancié pour les types de machine courants même lorsqu'aucune image de disquette n'est attachée, si bien que la surface est présente par défaut.

## Piloter le FDC

```c
// the FDC is programmed through I/O ports around 0x3f0-0x3f5; the guest writes a
// command byte followed by parameter bytes to the data register (0x3f5), then
// reads/writes data for read/write-style commands.
outb(0x3f5, FD_CMD);          // e.g. a read/write or format-track style command
outb(0x3f5, param);           // parameters advance the internal FIFO index
// the vulnerable pattern: a command whose data phase keeps writing into the fixed
// FIFO while the index is not reset/bounded -> write past the FIFO array (VENOM)
```

Le bug est que le contrôleur continue de stocker dans le buffer FIFO de taille fixe tout en avançant l'index au-delà des limites du tableau pour certaines commandes, si bien que l'invité réalise une écriture hors limites dans la mémoire de tas/struct environnante de QEMU, et une primitive de lecture correspondante.

## Notes d'exploitation

- Le contrôleur est présent pour les types de machine PC par défaut, que l'on configure ou non un lecteur ou une image de disquette, si bien que cette surface ne nécessite aucune configuration de VM inhabituelle.
- La primitive est une lecture et une écriture hors limites adjacentes à l'état du FDC dans le processus QEMU ; l'exploitation effectue un grooming du tas pour que des pointeurs utiles se trouvent à côté de la FIFO, puis les corrompt pour détourner le flux de contrôle.
- En tant que périphérique legacy à buffer unique, il est simple à piloter depuis l'invité avec des E/S de port brutes, ne nécessitant que la capacité d'exécuter des instructions d'E/S privilégiées.
- Les builds QEMU corrigés bornent l'index de la FIFO ; cela s'applique aux builds non corrigés, alors identifiez la version de QEMU.

## Références

- [Avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
- [VENOM (avis CrowdStrike)](https://www.crowdstrike.com/blog/venom-vulnerability-details/)
- [Awesome VM escape : FDC](https://github.com/WinMin/Awesome-VM-Exploit)
