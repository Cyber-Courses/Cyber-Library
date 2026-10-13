---
order: 4
title: "Périphériques audio : s'évader de VirtualBox via les contrôleurs de son émulés"
description: "VirtualBox émule les contrôleurs AC97, SB16 et Intel HD Audio, dont les pilotes invités programment des descripteurs de tampon et du DMA que le processus VM hôte lit pour véhiculer l'audio. Des failles dans la gestion des descripteurs de tampon et des longueurs DMA, en particulier dans le contrôleur Intel HDA, donnent un accès hors limites dans le processus hôte depuis la pile audio de l'invité."
keywords:
  - intel hda
  - ac97
  - sb16
  - audio
  - descripteur de tampon
---

# Périphériques audio

VirtualBox propose plusieurs cartes son émulées : le contrôleur Intel HD Audio (ICH6/HDA), l'AC97 et le SoundBlaster 16. Comme chez d'autres hyperviseurs, l'audio est une surface d'évasion facilement négligée mais bien réelle : le contrôleur HDA utilise une buffer descriptor list (BDL) en mémoire invitée décrivant les tampons DMA des données de flux, et l'hôte parcourt ces descripteurs pour transférer l'audio. Une longueur de descripteur ou une configuration DMA contrôlée par l'invité à laquelle l'hôte fait confiance est une primitive hors limites, et les cartes plus simples exposent une gestion de registres et de DMA relevant de la même classe de confiance sur les longueurs.

## La surface

```c
// Intel HDA: the guest programs stream descriptors and a BDL base; each BDL entry
// gives a guest address and length. The host walks the BDL to transfer stream data.
//  - a BDL entry length/count the host uses beyond the buffer -> OOB read/write
//  - stream position/registers driving an index the host does not bound
// AC97/SB16: DMA buffer registers and lengths with the same trusted-length class
```

```bash
lspci -nn | grep -i audio       # HDA / AC97 present
# an attacker guest audio driver programs the BDL and stream registers directly
```

## Notes d'exploitation

- Le contrôleur Intel HDA est la cible audio la plus riche en raison de son modèle DMA piloté par BDL ; une BDL forgée avec des longueurs auxquelles l'hôte fait confiance pilote un transfert hors limites dans le processus VM hôte.
- L'audio est fréquemment présent dans les configurations de VM de bureau par défaut, donc la surface existe sans configuration particulière ; confirmez le contrôleur avec `lspci`.
- L'invité contrôle la programmation des descripteurs et des registres depuis son pilote ; l'exploitation met en place directement les structures DMA plutôt que de lire de l'audio.
- Les primitives atterrissent dans le processus VM ; spécifiques à la version, à combiner avec une fuite. Le modèle HDA ressemble à ceux de QEMU et de VMware, donc la classe de bug se retrouve d'un hyperviseur à l'autre.

## Références

- [VirtualBox manual: audio](https://www.virtualbox.org/manual/)
- [Intel HD Audio specification](https://www.intel.com/content/www/us/en/standards/high-definition-audio-specification.html)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
