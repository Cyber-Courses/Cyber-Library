---
order: 4
title: "Tests de workflow des applications web : séquencement, concurrence, races et intégrité des paramètres métier"
description: "Logique métier et intégrité de l'état dans les flux web multi-étapes et multi-services : séquencement, concurrence et intégrité des paramètres."
keywords:
  - logique métier
  - sécurité des workflows
  - TOCTOU
  - idempotence
---

# Workflow

Le **workflow** couvre le **séquencement** (ordre des étapes et gates), les **effets parallèles** (races, double dépense, callbacks rejoués) et la **confiance** dans les paramètres numériques ou métier (montants, quotas) au fil de flux longs. Il suppose que les primitives de contrôle d'accès et d'injection sont comprises au niveau des nœuds frères.

## Structure (carte de la bibliothèque)

- **[Intégrité du flux](flow-integrity/index.md)** : séquencement vs alignement de l'état entre l'interface, les enregistrements et les sessions.
- **[Effets parallèles](parallel-effects/index.md)** : concurrence et handoff d'intégration (TOCTOU, rejeu, mauvais usage de l'idempotence).
- **[Frontières de confiance](trust-boundaries/index.md)** : intégrité des paramètres numériques et métier à travers les étapes.
