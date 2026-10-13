---
order: 6
title: "Périphériques audio : s'évader de QEMU via les cartes son émulées"
description: "QEMU émule des cartes son, le contrôleur Intel HD Audio, AC97, Sound Blaster 16 et ES1370, dont les pilotes invités programment des descripteurs de buffer et des registres que QEMU lit pour déplacer les données audio. Des failles dans la gestion des descripteurs de buffer et le traitement des longueurs DMA, notamment dans le contrôleur Intel HDA, donnent un accès hors limites dans le processus QEMU."
keywords:
  - intel hda
  - ac97
  - sound blaster
  - audio
  - buffer descriptor
---

# Périphériques audio

L'audio est une surface d'évasion facilement négligée. QEMU émule plusieurs cartes son, le contrôleur Intel HD Audio (`intel-hda`), AC97, Sound Blaster 16 et ES1370, chacune pilotée par des écritures de registre de l'invité et, pour le HDA, des listes de descripteurs de buffer en mémoire invité. Le contrôleur HDA utilise un anneau de descripteurs de buffer (une BDL) décrivant des buffers DMA pour les données de flux, et QEMU parcourt ces descripteurs pour déplacer l'audio, si bien qu'une longueur de descripteur ou une configuration DMA contrôlée par l'invité à laquelle QEMU se fie est une primitive hors limites. Les cartes plus simples exposent une gestion des registres et du DMA avec la même classe de bugs de confiance en la longueur.

## La surface

```c
// Intel HDA: the guest programs stream descriptors and a buffer descriptor list
// (BDL) base; each BDL entry gives a guest address and length for a DMA buffer.
// QEMU walks the BDL to transfer stream data. Primitives:
//  - a BDL entry length/count QEMU uses for a transfer beyond the buffer -> OOB
//  - stream/position registers driving an index the model does not bound
// AC97/SB16/ES1370: DMA buffer registers and lengths with the same trusted-length class
```

```bash
lspci -nn | grep -i audio       # intel-hda / AC97 / ES1370 present
# the guest audio driver programs the controller; an attacker driver sets the BDL
# and stream registers directly with controlled addresses and lengths
```

## Notes d'exploitation

- Le contrôleur Intel HDA est la cible audio la plus riche en raison de son modèle DMA à liste de descripteurs de buffer ; une BDL élaborée avec des longueurs auxquelles QEMU se fie pilote un transfert hors limites dans QEMU.
- Les périphériques audio sont fréquemment présents dans les configurations de VM de type bureau par défaut, si bien que la surface existe sans réglage particulier ; confirmez avec `lspci`.
- L'invité contrôle la programmation des descripteurs et des registres depuis son pilote ; l'exploitation met en place les structures DMA directement plutôt que de jouer du son normalement.
- Les primitives atterrissent dans le processus QEMU ; spécifiques à une version de QEMU, à associer à une fuite et à un grooming.

## Références

- [Documentation audio QEMU](https://www.qemu.org/docs/master/system/devices/)
- [Spécification Intel High Definition Audio](https://www.intel.com/content/www/us/en/standards/high-definition-audio-specification.html)
- [Avis de sécurité QEMU](https://www.qemu.org/docs/master/system/security.html)
