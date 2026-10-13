---
title: "Effets parallèles dans les workflows web : races, double dépense, rejeu de webhook et mauvais usage de l'idempotence"
description: "Concurrence et handoff d'intégration dans les workflows : races, double dépense, callbacks rejoués et mauvais usage de l'idempotence."
keywords:
  - TOCTOU
  - race condition
  - rejeu de webhook
  - idempotence
---

# Effets parallèles

Les effets parallèles couvrent ce qui se produit lorsque deux requêtes ou plus touchent le même état de workflow, ou lorsque des callbacks asynchrones arrivent dans le désordre. Ils côtoient le [séquencement](../flow-integrity/sequencing/index.md) (ordre des étapes visibles par l'utilisateur) et l'[alignement de l'état](../flow-integrity/state-alignment/index.md) (ce que croit chaque couche).

## Concurrence

| Page | Objet |
|------|--------|
| [TOCTOU dans les workflows](toctou.md) | Écarts check-then-act entre requêtes |
| [Double dépense](double-spend.md) | Dépenser deux fois le même solde, coupon ou crédit |
| [Requêtes parallèles](parallel-requests.md) | Fan-out intentionnel de requêtes contre des verrous faibles |

## Handoff d'intégration

| Page | Objet |
|------|--------|
| [Rejeu de webhook](webhook-replay.md) | Livraison en double de callbacks signés ou non signés |
| [Callback hors ordre](out-of-order-callback.md) | Événements de paiement ou de provisioning appliqués dans le mauvais ordre |
| [Mauvais usage des clés d'idempotence](idempotency-key-misuse.md) | Clés ignorées, mal portées ou réinitialisées à la retentative |
