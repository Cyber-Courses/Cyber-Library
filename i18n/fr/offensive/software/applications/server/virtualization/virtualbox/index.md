---
order: 3
title: "VirtualBox : attaquer l'hyperviseur de bureau d'Oracle"
description: "VirtualBox est un hyperviseur de type 2 exécutant chaque VM à l'intérieur d'un processus VBoxSVC/VirtualBox sur l'hôte. Les évasions guest-to-host ciblent ses périphériques émulés, l'accélération 3D, l'audio, le réseau et l'USB, ainsi que l'intégration Guest Additions (dossiers partagés et presse-papiers via HGCM). C'est une cible fréquente de recherche et de Pwn2Own, dotée d'une large surface de périphériques."
keywords:
  - virtualbox
  - vboxsvc
  - guest additions
  - hgcm
  - hyperviseur de type 2
---

# VirtualBox

VirtualBox est l'hyperviseur de bureau multiplateforme d'Oracle. Chaque VM s'exécute à l'intérieur d'un processus hôte (le frontend `VirtualBox`/`VBoxHeadless` avec `VBoxSVC`), qui émule les périphériques de la VM, de sorte qu'une évasion guest-to-host corrompt ce processus et s'exécute sur l'hôte de l'utilisateur. Sa surface de périphériques est large, accélération 3D, plusieurs modèles audio et réseau, et contrôleurs USB, et les Guest Additions ajoutent les dossiers partagés et le presse-papiers/glisser-déposer via le Host-Guest Communication Manager (HGCM), une surface d'intégration proéminente. VirtualBox est une cible récurrente de recherche et de Pwn2Own précisément en raison de cette étendue.

```bash
# from the guest: emulated devices and Guest Additions
lspci -nn; lsusb; lsmod | grep -i vbox
VBoxControl --version 2>/dev/null         # Guest Additions present
```

## Sous-sujets

- **[Évasion guest-to-host](guest-to-host-escape/index.md)** : sortir via les périphériques émulés.
- **[Abus des dossiers partagés et du presse-papiers](shared-folder-and-clipboard-abuse.md)** : l'intégration HGCM des Guest Additions.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs récurrents de périphériques et de HGCM.

## Références

- [VirtualBox documentation](https://www.virtualbox.org/manual/)
- [Oracle critical patch updates (VirtualBox)](https://www.oracle.com/security-alerts/)
- [Zero Day Initiative: VirtualBox research](https://www.zerodayinitiative.com/blog)
