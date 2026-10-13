---
order: 3
title: "Contrôleurs USB : s'évader de VirtualBox via les OHCI, EHCI et xHCI émulés"
description: "VirtualBox émule les contrôleurs hôtes USB OHCI, EHCI et xHCI dont les pilotes invités construisent des descripteurs de transfert et des anneaux en mémoire invitée que le processus VM hôte parcourt par DMA. Des failles dans la gestion des chaînes de descripteurs et des anneaux et contextes de périphériques xHCI donnent un accès hors limites dans le processus hôte, une surface d'évasion VirtualBox récurrente, y compris à Pwn2Own."
keywords:
  - contrôleur usb
  - xhci
  - ehci
  - ohci
  - descripteur de transfert
---

# Contrôleurs USB

VirtualBox émule les contrôleurs hôtes OHCI (USB 1.1), EHCI (USB 2.0) et xHCI (USB 3.0). Le pilote invité programme un contrôleur via des registres et construit ses structures de données, descripteurs d'endpoint et de transfert pour OHCI/EHCI, et les anneaux de commandes, d'événements et de transfert de TRB pour xHCI, en mémoire invitée, que le processus VM hôte lit par DMA pour effectuer les transferts. Parcourir ces structures construites par l'invité est la surface d'évasion, et le modèle xHCI en particulier a été un vecteur d'évasion VirtualBox, y compris à Pwn2Own.

## La surface

```c
// xHCI: the guest sets ring base registers and writes TRBs (type, flags, ptr, len).
// the host walks the command/transfer rings following link TRBs and tracks per-slot
// device contexts and endpoints. Primitives:
//  - a link TRB forming a cycle or pointing out of range -> over-read when walked
//  - a transfer length the host uses for a copy beyond the mapped buffer
//  - a slot/endpoint index used to index context arrays without bounds -> OOB/UAF
// OHCI/EHCI: endpoint/transfer descriptor lists with the same descriptor-walk classes
```

```bash
lspci -nn | grep -i usb       # OHCI/EHCI/xHCI present (xHCI needs the extension pack era config)
# an attacker guest USB driver programs the rings/descriptors directly
```

## Notes d'exploitation

- xHCI est la cible la plus riche car l'hôte suit un état par-slot et par-endpoint important, offrant des opportunités de type-confusion et de use-after-free à la mise en place/démantèlement des contextes, en plus des over-reads lors du parcours des anneaux.
- L'invité contrôle la base et le contenu des anneaux/descripteurs depuis son pilote ; l'exploitation programme directement le contrôleur plutôt que d'attacher un périphérique normalement.
- Les mêmes modèles de contrôleurs apparaissent dans QEMU et VMware, donc les bugs xHCI se retrouvent d'un hyperviseur à l'autre ; voir [contrôleurs USB QEMU](../../kvm/qemu/guest-to-host-escape/usb-controllers.md).
- Les primitives atterrissent dans le processus VM ; spécifiques à la version, à combiner avec une fuite.

## Références

- [xHCI specification](https://www.intel.com/content/dam/www/public/us/en/documents/technical-specifications/extensible-host-controller-interface-usb-xhci.pdf)
- [Zero Day Initiative: VirtualBox USB research](https://www.zerodayinitiative.com/blog)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
