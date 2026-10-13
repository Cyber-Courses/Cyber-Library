---
title: "Abus des workflows d'annulation, de chargeback et de remboursement : race avec le règlement et crédits en double"
description: "Transactions compensatoires qui créditent les comptes avant que les débits soient réglés, ou qui autorisent plusieurs remboursements pour un seul débit."
keywords:
  - workflow
  - remboursement
---

# Abus des workflows d'annulation

## Contexte

Les API de remboursement peuvent ne pas être idempotentes ; des requêtes de remboursement et de capture parallèles peuvent créditer en double lorsque les mises à jour du ledger ne sont pas atomiques. Les webhooks de chargeback peuvent être rejoués si les identifiants d'événement ne sont pas stockés.

## Outils

- **Burp Suite Turbo Intruder** : single-packet attack pour déclencher des requêtes de remboursement et de capture concurrentes.
- **Burp Repeater** : groupes d'onglets pour envoyer des requêtes de remboursement groupées en parallèle, et pour rejouer les callbacks de chargeback.
- **curl** : requêtes de remboursement parallèles (en arrière-plan ou pilotées avec xargs).

## Références

- PortSwigger Web Security Academy: Race conditions
- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
