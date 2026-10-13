---
title: "Intégrité du flux dans les applications web : séquencement versus alignement de l'état entre l'interface, l'enregistrement et la session"
description: "Cohérence du workflow : séquencement des étapes versus alignement de l'état entre les clients, les enregistrements et les sessions."
keywords:
  - intégrité du flux
  - logique métier
  - workflow
---

# Intégrité du flux

L'**intégrité du flux** interroge la cohérence de l'**histoire** d'un workflow avant d'analyser les races ou l'altération de valeurs numériques. Le planificateur la divise en **séquencement** (ordre, approbations, annulations) et **alignement de l'état** (ce que croient respectivement l'interface, la ligne en base de données et la session).

## Sous-thèmes

- **[Séquencement](sequencing/index.md)** : saut d'étape, accès hors ordre, contournement d'approbation, omission de paramètre, renouvellement de droits, workflows d'annulation.
- **[Alignement de l'état](state-alignment/index.md)** : interface vs enregistrement vs points de contrôle de session.
- **[Effets parallèles](../parallel-effects/index.md)** : races, double dépense, rejeu de callback (nœud frère sous workflow).

**Frontières de confiance** pour les paramètres numériques et métier : **[Frontières de confiance](../trust-boundaries/index.md)**.

## Outils

- **Burp Suite (Repeater)** : rejouer et réordonner les étapes de workflow capturées pour sonder le séquencement et l'alignement de l'état.
- Tests manuels avec Burp Repeater et payloads sur mesure.

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
