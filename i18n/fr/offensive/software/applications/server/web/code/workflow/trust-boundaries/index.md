---
title: "Frontières de confiance dans les workflows web : falsification des prix, SKU, niveaux et paramètres métier inter-étapes"
description: "Les paramètres numériques et métier qui traversent les étapes (montants, quantités, rôles) doivent rester cohérents avec la politique appliquée côté serveur."
keywords:
  - logique métier
  - falsification de paramètres
---

# Frontières de confiance

Les frontières de confiance sont les **limites** où une donnée quitte un contexte d'application des règles pour entrer dans un autre : du navigateur à l'API, du microservice A au B, ou d'un formulaire synchrone à un job asynchrone. La bibliothèque se concentre sur les paramètres **numériques** et **métier** faciles à falsifier lorsque des champs cachés ou des claims JWT sont crus aveuglément.

## Pages

| Page | Objet |
|------|--------|
| [Intégrité numérique](numeric-integrity.md) | Prix, frais, lignes de taxe, quantités |
| [Paramètres métier](business-parameters.md) | SKU produit, niveau de livraison, niveau de droits |

## Outils

- **Burp Suite (Repeater)** : falsifier les paramètres numériques et métier au moment où ils traversent les étapes, puis rejouer pour valider.
- Tests manuels avec Burp Repeater et des payloads conçus à la main.

## Références

- PortSwigger Web Security Academy : Business logic vulnerabilities
- OWASP WSTG : Testing for Business Logic
