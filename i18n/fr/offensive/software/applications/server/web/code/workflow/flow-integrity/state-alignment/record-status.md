---
title: "Statut d'enregistrement vs effets de bord : indicateurs de workflow périmés, jobs en double et règlement incohérent dans les applications web"
description: "Colonnes de statut en base de données qui sont en retard sur des actions irréversibles (email envoyé, stock décrémenté, paiement capturé)."
keywords:
  - workflow
  - cohérence
---

# Statut d'enregistrement

## Contexte

Une ligne reste `PENDING` alors que des webhooks ont déjà expédié des marchandises. Les jobs « sync status » rejoués peuvent expédier en double ou refacturer parce que l'action compensatoire s'appuie uniquement sur `status`, et non sur un registre d'effets.

## Théorie

Modélisez les effets de bord idempotents avec des clés métier uniques et des journaux d'événements en append-only, et non avec un unique indicateur mutable.

## Pratique

- Déclenchez deux fois le bouton admin « retry provisioning » en staging tout en surveillant les emails en double ou les clés de licence en double.

## Outils

- Logs applicatifs et requêtes sur la base de staging

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
