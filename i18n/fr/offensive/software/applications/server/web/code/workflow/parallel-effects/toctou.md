---
title: "TOCTOU dans les workflows web : races time-of-check to time-of-use sur les soldes, quotas et approbations"
description: "Écarts time-of-check to time-of-use lorsqu'un workflow lit un état, branche, puis agit sans tenir un verrou cohérent."
keywords:
  - TOCTOU
  - race condition
---

# TOCTOU

## Contexte

L'application charge un enregistrement (quota, solde, indicateur d'approbation), le valide en mémoire, puis écrit une mise à jour dans une seconde étape. Une autre requête modifie l'enregistrement entre la lecture et l'écriture. Lors d'évaluations, démontrez l'impact uniquement sur des données hors production et avec un périmètre écrit pour les tests de concurrence.

## Théorie

Une isolation faible se manifeste lorsque la validation et la mutation sont des gestionnaires HTTP séparés ou utilisent des transactions différentes sans `SELECT … FOR UPDATE` ou équivalent.

## Pratique

- Dans un lab, émettez deux requêtes concurrentes qui passent toutes deux un contrôle « usage unique » ; observez les effets en double dans les logs ou la base de staging.

## Outils

- **Burp Suite** (Turbo Intruder ou onglets Repeater parallèles)
- **scripts personnalisés** (httpx, curl parallèle)

## Références

- PortSwigger Web Security Academy: Race conditions
- OWASP WSTG: Testing for Business Logic
