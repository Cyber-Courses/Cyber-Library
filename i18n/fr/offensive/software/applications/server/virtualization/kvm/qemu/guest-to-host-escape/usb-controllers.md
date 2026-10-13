---
order: 3
title: "Contrôleurs USB : s'évader de QEMU via les UHCI, EHCI et XHCI émulés"
description: "QEMU émule des contrôleurs hôtes USB dont les pilotes invités construisent des descripteurs de transfert et des anneaux en mémoire invité que QEMU parcourt pour effectuer les transferts. Des failles dans l'analyse de la chaîne de descripteurs, dans le modèle d'anneaux et de contextes de périphérique XHCI, et dans la gestion des périphériques USB émulés donnent un accès hors limites dans le processus QEMU depuis la pile USB de l'invité."
keywords:
  - usb controller
  - xhci
  - ehci
  - uhci
  - transfer descriptor
---

# Contrôleurs USB

QEMU émule les contrôleurs hôtes UHCI et EHCI (USB 1.1/2.0) et XHCI (USB 3.0), ainsi qu'une gamme de périphériques USB émulés. Le pilote invité programme un contrôleur via des registres MMIO/PIO et construit les structures de données du contrôleur, descripteurs de transfert et queue heads pour UHCI/EHCI, ou les anneaux de commande, d'événement et de transfert de TRB pour XHCI, en mémoire invité, que QEMU lit pour effectuer les transferts. Le parcours de ces structures construites par l'invité, et la gestion des périphériques USB émulés qui se trouvent derrière, est la surface d'évasion, et le modèle XHCI en particulier a produit des bugs de sûreté mémoire.

## Piloter le contrôleur

```bash
lspci -nn | grep -i usb        # UHCI/EHCI/XHCI model present
# the guest xhci/ehci/uhci driver (or an attacker driver) programs the ring/queue
# base registers and writes descriptors/TRBs with controlled fields
```

```c
// XHCI: the guest sets ring base registers and writes TRBs (type, flags, ptr, len).
// QEMU walks the command/transfer rings following link TRBs and tracks per-slot
// device contexts and endpoints. Primitives:
//  - a link TRB forming a cycle or pointing out of range -> over-read when walked
//  - a transfer length QEMU uses for a copy beyond the mapped buffer
//  - a slot/endpoint index used to index context arrays without bounds -> OOB/UAF
// UHCI/EHCI: frame list + queue-head/transfer-descriptor chains with the same classes
```

## Notes d'exploitation

- XHCI est la cible la plus riche car QEMU suit un état important par slot et par endpoint, offrant des opportunités de confusion de type et d'use-after-free lors de la configuration et de la destruction de contexte, en plus des over-reads lors du parcours des anneaux.
- Les périphériques USB émulés (derrière le contrôleur) analysent leur propre protocole influencé par l'invité et constituent une surface secondaire.
- Comme avec les autres modèles de périphériques, l'invité contrôle la base et le contenu de l'anneau/descripteur depuis son pilote ; l'exploitation programme le contrôleur directement.
- Les primitives atterrissent dans le tas du processus QEMU ; spécifiques à une version de QEMU, à associer à une fuite. Les mêmes modèles de contrôleur apparaissent dans d'autres hyperviseurs, donc les bugs peuvent se transposer.

## Références

- [Documentation USB QEMU](https://www.qemu.org/docs/master/system/devices/usb.html)
- [Avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
- [Awesome VM escape : USB](https://github.com/WinMin/Awesome-VM-Exploit)
