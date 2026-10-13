---
order: 2
title: "Contrôleurs USB : s'évader d'ESXi via les UHCI, EHCI et XHCI émulés"
description: "ESXi émule des contrôleurs hôtes USB (UHCI, EHCI, XHCI) dont les pilotes invités construisent des descripteurs de transfert et de file en mémoire invitée que le processus vmx lit et parcourt par DMA. Des failles dans l'analyse des chaînes de descripteurs, la gestion des anneaux ou le traitement des requêtes de périphérique donnent un accès hors limites dans le processus hôte, une évasion guest-to-host pilotée entièrement depuis la pile USB de l'invité."
keywords:
  - contrôleur usb
  - xhci
  - ehci
  - uhci
  - descripteur de transfert
---

# Contrôleurs USB

Un invité voit un ou plusieurs contrôleurs hôtes USB émulés : UHCI et EHCI pour l'USB 1.1/2.0, et XHCI pour l'USB 3.0. Le pilote invité programme chaque contrôleur via des registres MMIO ou d'E/S et construit des structures de données en mémoire invitée, descripteurs de transfert et queue heads pour UHCI/EHCI, et les anneaux de commandes, d'événements et de transfert avec leurs TRB pour XHCI, que le modèle de contrôleur côté hôte dans `vmx` lit par DMA et parcourt. Analyser ces structures construites par l'invité est la surface d'évasion : une chaîne de descripteurs ou un anneau que l'hôte parcourt avec une longueur à laquelle il fait confiance ou un lien pendant donne un accès hors limites dans `vmx`.

## Piloter le contrôleur depuis l'invité

```bash
# identify the emulated controllers
lspci -nn | grep -i usb        # Intel UHCI/EHCI, or an XHCI controller
# the guest xhci/ehci driver programs the controller; from an attacker driver you
# control the register programming and the descriptor/ring contents directly
```

Pour XHCI, les structures clés construites par l'invité sont l'anneau de commandes, l'anneau d'événements et les anneaux de transfert par endpoint, chacun une séquence de Transfer Request Blocks (TRB) avec un type, des flags et un pointeur de données. Le modèle hôte lit la base de l'anneau depuis un registre que l'invité positionne, puis parcourt les TRB en suivant les link TRB :

```c
// a malicious guest points a ring base register at attacker memory and writes
// TRBs with: an oversized transfer length, a link TRB forming a cycle or pointing
// out of range, or a device-context/endpoint index the host indexes without bounds
// checking -> OOB read/write when vmx walks the ring or copies the transfer
```

Pour UHCI/EHCI, l'équivalent est la frame list et les chaînes queue-head/descripteur de transfert, où une chaîne forgée (boucles, pointeurs hors plage, longueurs incohérentes) pilote le parcours de l'hôte hors limites.

## Notes d'exploitation

- Le modèle de contrôleur fait confiance à la base de l'anneau/descripteur et aux pointeurs de lien que l'invité programme ; un lien formant un cycle ou pointant hors de la RAM invitée, ou une longueur de TRB que l'hôte utilise pour une copie, est la primitive typique.
- XHCI est la cible moderne à forte valeur car son modèle d'anneaux et de contextes de périphériques est complexe et l'hôte doit suivre un état par-endpoint important, offrant des opportunités de type-confusion et de use-after-free au démantèlement des endpoints et des contextes.
- Les descripteurs USB des périphériques émulés (renvoyés à l'invité) sont une surface distincte pour l'influence host-to-guest, mais la direction d'évasion ici est celle des anneaux construits par l'invité et lus par l'hôte.
- Comme pour les autres périphériques, un bug spécifique au build plus une fuite d'ASLR sont nécessaires ; identifiez la version d'ESXi.

## Références

- [xHCI specification](https://www.intel.com/content/dam/www/public/us/en/documents/technical-specifications/extensible-host-controller-interface-usb-xhci.pdf)
- [Zero Day Initiative: VMware USB research](https://www.zerodayinitiative.com/blog)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
