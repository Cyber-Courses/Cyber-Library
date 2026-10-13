---
title: "Double dépense dans les applications web : coupons, crédits de portefeuille, jetons à usage unique et réservations de stock"
description: "Appliquer plus d'une fois le même crédit, coupon, jeton de remboursement ou réservation de stock à travers des requêtes parallèles ou rejouées."
keywords:
  - double dépense
  - abus de coupon
---

# Double dépense

## Contexte

La « double dépense » en test web désigne la réutilisation d'un unique artefact d'autorisation (code promo, solde de portefeuille, jeton à usage unique) sur deux validations réussies. Elle diffère de la double dépense des réseaux de paiement ; ici le bug relève de la logique de ledger ou de coupon au niveau applicatif.

## Théorie

Échoue lorsque la consommation n'est pas atomique avec le débit, ou lorsque les clés d'idempotence manquent pour les soumissions rejouées.

## Pratique

- Préparez une unique consommation non encore utilisée (un coupon, un jeton ou une réservation neufs n'ayant jamais été appliqués), capturez la requête de consommation sans l'envoyer, puis expédiez plusieurs copies identiques de façon concurrente (single-packet attack de Turbo Intruder, ou connexions parallèles). L'objectif est que toutes les copies passent le contrôle « est-ce encore valide ? » avant que l'une d'elles ne marque l'artefact comme consommé. Rejouer une consommation déjà terminée ne teste que la réutilisation post-validation ; la race exige que les doublons arrivent tant que l'artefact est encore non consommé.

## Outils

- **Burp Suite**
- **OWASP ZAP**

## Références

- PortSwigger Web Security Academy: Race conditions
- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
