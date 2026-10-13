---
order: 1
title: "Évasion guest-to-host : sortir d'une machine virtuelle VirtualBox"
description: "Un invité VirtualBox s'évade en corrompant le processus VM hôte qui émule ses périphériques. La surface accessible est l'ensemble des périphériques émulés : le chemin d'accélération 3D, les contrôleurs audio, les cartes réseau et les contrôleurs USB, chacun analysant des écritures de registres et des structures DMA pilotées par l'invité, plus les services HGCM derrière les Guest Additions."
keywords:
  - évasion virtualbox
  - émulation de périphériques
  - accélération 3d
  - usb
  - guest-to-host
---

# Évasion guest-to-host

Une VM VirtualBox s'exécute à l'intérieur d'un processus hôte qui émule ses périphériques, donc s'évader revient à faire manipuler incorrectement par ce processus des données contrôlées par l'invité. La surface des périphériques est large : le chemin d'accélération 3D (utilisé par les Guest Additions pour les graphismes accélérés), les contrôleurs audio émulés, les cartes réseau et les contrôleurs USB, chacun lisant côté hôte les écritures de registres et les descripteurs DMA de l'invité. L'exécution de code atterrit dans le processus VM, qui s'exécute avec les privilèges de l'utilisateur ayant démarré la VM. L'intégration Guest Additions (dossiers partagés, presse-papiers) ajoute la surface des services HGCM, traitée séparément.

```bash
# the emulated devices visible in the guest (escape surface)
lspci -nn; lsusb
dmesg | grep -iE 'e1000|pcnet|ac97|hda|ohci|ehci|xhci'
```

## Sous-sujets

- **[Accélération 3D](3d-acceleration.md)** : le chemin de commandes des graphismes accélérés.
- **[Périphériques audio](audio-devices.md)** : les contrôleurs AC97 et HD Audio émulés.
- **[Cartes réseau](network-adapters.md)** : les modèles e1000, PCnet et virtio-net.
- **[Contrôleurs USB](usb-controllers.md)** : les contrôleurs OHCI, EHCI et xHCI.

## Références

- [VirtualBox manual: virtual hardware](https://www.virtualbox.org/manual/)
- [Zero Day Initiative: VirtualBox escapes](https://www.zerodayinitiative.com/blog)
- [Oracle security alerts](https://www.oracle.com/security-alerts/)
