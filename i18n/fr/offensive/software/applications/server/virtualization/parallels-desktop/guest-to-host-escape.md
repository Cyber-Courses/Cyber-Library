---
order: 1
title: "Évasion guest-to-host : sortir d'une machine virtuelle Parallels"
description: "Un invité Parallels s'évade en corrompant les processus Parallels côté hôte qui émulent les périphériques de la VM et desservent les canaux invité-hôte. La surface accessible est les périphériques émulés, graphiques, réseau, USB et stockage, et les interfaces de communication de Parallels Tools, une faille donnant l'exécution de code sur l'hôte macOS au privilège du processus Parallels."
keywords:
  - parallels escape
  - device emulation
  - prl
  - macos
  - guest-to-host
---

# Évasion guest-to-host

Les périphériques et services d'intégration d'une VM Parallels sont émulés dans des processus `prl_*` côté hôte sur macOS, si bien qu'une évasion guest-to-host fait qu'un de ces processus traite mal des données contrôlées par l'invité. La surface comporte deux parties : les modèles de périphériques émulés (graphiques y compris la 3D accélérée, l'adaptateur réseau, les contrôleurs USB et le stockage), qui analysent les écritures de registre et les structures DMA de l'invité, et les interfaces de communication de Parallels Tools, qui analysent les requêtes d'intégration de l'invité. L'exécution de code atterrit dans un processus hôte Parallels sur macOS, au privilège de ce processus.

```bash
# the emulated device inventory in the guest (escape surface)
lspci -nn 2>/dev/null; lsusb 2>/dev/null
dmesg 2>/dev/null | grep -i prl
```

## La surface

```text
Guest-to-host escape surface:
- graphics / 3D acceleration: a guest-controlled command/surface stream parsed host-side
  (the richest device surface, as on other hypervisors' 3D paths)
- network adapter: TX/RX descriptor and offload handling
- USB controllers: transfer-descriptor/ring handling
- storage controller: command and DMA descriptor handling
- Parallels Tools channels: integration requests (see shared-folder/clipboard abuse)
```

Les classes de bugs du modèle de périphériques sont les classes standard (longueurs supposées fiables, index non bornés, use-after-free sur les objets de requête), et le chemin 3D/graphique est typiquement le plus productif, comme sur d'autres hyperviseurs. L'analyse des canaux Tools est la surface d'intégration spécifique à Parallels.

## Notes d'exploitation

- L'exécution de code atterrit dans un processus hôte Parallels sur macOS ; son privilège détermine l'impact immédiat, avec une possible élévation de privilèges locale supplémentaire sur macOS pour le contrôle total de l'hôte.
- Le chemin graphique/3D est le point chaud récurrent du périphérique ; les canaux de Parallels Tools sont le point chaud d'intégration, analogues au GuestRPC de VMware et au HGCM de VirtualBox.
- L'invité pilote les modèles de périphériques et les canaux Tools depuis des pilotes/clients contrôlés, postant des structures élaborées directement.
- Les évasions Parallels sont spécifiques à une version ; identifiez la version de Parallels Desktop et faites-la correspondre à l'avis de sécurité, et voir la surface d'intégration [dossier partagé et presse-papiers](shared-folder-and-clipboard-abuse.md).

## Références

- [Documentation Parallels Desktop](https://www.parallels.com/products/desktop/)
- [Zero Day Initiative : évasions Parallels Pwn2Own](https://www.zerodayinitiative.com/blog)
