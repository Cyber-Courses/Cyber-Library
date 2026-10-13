---
order: 5
title: "Parallels Desktop : attaquer l'hyperviseur de bureau macOS"
description: "Parallels Desktop exécute des VM sur macOS, avec l'émulation de périphériques et les services d'intégration dans des processus côté hôte. Les évasions guest-to-host ciblent ses périphériques émulés et l'intégration Parallels Tools, dossiers partagés, presse-papiers et glisser-déposer sur les canaux de communication invité-hôte de Parallels. C'est une cible Pwn2Own récurrente sur l'hôte macOS."
keywords:
  - parallels desktop
  - macos
  - parallels tools
  - shared folders
  - prl
---

# Parallels Desktop

Parallels Desktop est le principal hyperviseur de bureau macOS, exécutant des VM Windows et Linux avec l'émulation de périphériques et les services d'intégration hébergés dans les propres processus de Parallels (la famille `prl_*`) sur l'hôte macOS. Une évasion guest-to-host corrompt un de ces composants côté hôte et s'exécute sur macOS. La surface d'attaque est l'ensemble des périphériques émulés et, au premier plan, l'intégration Parallels Tools, les dossiers partagés, le presse-papiers partagé et le glisser-déposer, transportés sur les canaux de communication invité-hôte de Parallels. Parallels est une cible Pwn2Own récurrente, avec des évasions trouvées à plusieurs reprises à la fois dans les modèles de périphériques et dans l'intégration Tools.

```bash
# from the guest: emulated devices and Parallels Tools
lspci -nn 2>/dev/null; lsusb 2>/dev/null         # Linux guest
# Parallels Tools present (guest integration drivers/services)
lsmod 2>/dev/null | grep -i prl
```

## Sous-thèmes

- **[Évasion guest-to-host](guest-to-host-escape.md)** : s'évader via les périphériques émulés.
- **[Abus des dossiers partagés et du presse-papiers](shared-folder-and-clipboard-abuse.md)** : la surface d'intégration Parallels Tools.
- **[Exploits d'évasion connus](known-escape-exploits.md)** : les bugs récurrents des périphériques et de Tools.

## Références

- [Documentation Parallels Desktop](https://www.parallels.com/products/desktop/)
- [Zero Day Initiative : recherche Parallels](https://www.zerodayinitiative.com/blog)
- [Mises à jour de sécurité Parallels](https://www.parallels.com/products/desktop/)
