---
title: "Webhooks et callbacks hors ordre : bugs d'ordre causal dans les intégrations de paiement et de provisioning"
description: "Événements asynchrones appliqués dans l'ordre d'arrivée alors que le contrat métier exige un ordre causal (par exemple autoriser avant capturer)."
keywords:
  - webhook
  - ordonnancement
---

# Callbacks hors ordre

## Contexte

Deux callbacks référencent la même clé métier mais arrivent sur des connexions différentes. Si le gestionnaire applique un « last write wins » sans version d'état monotone, la capture peut être traitée avant l'autorisation, ou un remboursement avant qu'un débit soit réglé, laissant les comptes incohérents.

## Théorie

Comparez avec les version vectors des systèmes distribués ; beaucoup d'applications CRUD les omettent dans les gestionnaires de webhook.

## Pratique

- En staging, retardez artificiellement un callback (proxy lent) et envoyez d'abord l'événement dépendant ; observez les tables de ledger ou de droits.

## Outils

- **Burp Suite** avec **Throttling**
- **tc** / **netem** dans un réseau de lab

## Références

- PortSwigger Web Security Academy: Race conditions
- OWASP WSTG: Testing for Business Logic
