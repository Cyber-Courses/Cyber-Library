---
title: "Accès conditionnel : passer à travers les politiques de connexion"
order: 5
description: "Contournement des politiques d'accès conditionnel Entra : usurpation de l'agent utilisateur et de la plateforme d'appareil, protocoles d'authentification héritée, et failles de localisation ou d'appareil conforme."
keywords:
  - accès conditionnel
  - contournement
  - authentification héritée
  - conformité des appareils
  - faille de politique
---

# Accès conditionnel

Les politiques d'accès conditionnel conditionnent la connexion à certains critères (application, plateforme, localisation, état de l'appareil), et les contournements proviennent des interstices entre ces critères. Si une politique ne vise que certaines applications ou plateformes, vous vous présentez comme quelque chose qu'elle ne couvre pas ; si elle fait confiance à un signal de conformité d'appareil ou de localisation, vous l'usurpez ou vous vous connectez depuis celui-ci.

## Trouver les failles et s'y glisser

```bash
# present a device platform / client the policy does not cover by changing the user-agent
# (e.g. a policy that blocks mobile but not a spoofed desktop or legacy client)
curl -s -A "Mozilla/5.0 (Windows NT 10.0)" https://login.microsoftonline.com/...

# legacy-auth endpoints (Exchange ActiveSync, Autologon) often fall outside modern-auth CA
```

Énumérez les politiques une fois que vous détenez un jeton (`roadrecon` extrait les politiques d'accès conditionnel) et examinez-les pour repérer l'application, la plateforme ou le type de client non couvert.

## Notes d'exploitation

- Les protocoles d'authentification héritée sont la faille classique : de nombreuses politiques ne s'appliquent qu'à l'authentification moderne, de sorte que les points de terminaison en authentification de base les contournent.
- Une politique exigeant un appareil conforme ou joint de façon hybride est déjouée en enregistrant ou en forgeant un tel appareil (voir [enregistrement d'appareil](../devices/device-registration.md)).
- Les politiques basées sur la localisation font confiance à l'adresse IP source ; un VPN ou une sortie dans une plage autorisée les satisfait.

## Outils

- **ROADtools** (`roadrecon`) : extraire et analyser les politiques d'accès conditionnel.
- **AADInternals** / **TokenTactics** : s'authentifier via des points de terminaison alternatifs.

## Références

- [dirkjanm.io : recherche sur l'accès conditionnel](https://dirkjanm.io/)
- [ROADtools](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud : accès conditionnel](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
