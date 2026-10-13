---
title: "Contournement du renouvellement de droits : réutilisation d'essais, de coupons ou de licences expirés à travers les flux de renouvellement"
description: "Workflows où les chemins de renouvellement, de mise à niveau ou de resoumission sautent le paiement ou la revérification exigés lors de l'inscription initiale."
keywords:
  - workflow
  - logique métier
---

# Contournement du renouvellement de droits

## Contexte

Les endpoints de renouvellement ou d'extension peuvent supposer que le compte est déjà de confiance et sauter les contrôles de carte ou d'identité exigés lors de l'achat initial. Les attaquants effectuent une rétrogradation puis un renouvellement pour récupérer des avantages sans satisfaire tous les critères.

## Outils

- **Burp Suite (Repeater)** : rejouer les requêtes de renouvellement et de mise à niveau pour supprimer les étapes de revérification ou de paiement.
- Tests manuels avec Burp Repeater et payloads sur mesure.

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
