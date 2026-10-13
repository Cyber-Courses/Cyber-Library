---
order: 2
title: "Falsification des paramètres métier : SKU, niveau d'offre, livraison et codes de rôle dans les requêtes multi-étapes"
description: "Codes de SKU, de niveau de service, de région ou de rôle transportés d'une étape à l'autre, que le serveur devrait résoudre à partir de l'identité authentifiée et du service d'inventaire."
keywords:
  - logique métier
  - falsification de SKU
---

# Paramètres métier

## Contexte

Les workflows transmettent productId, plan ou shippingMethod sous forme de champs cachés. Si le traitement de la commande lit ces champs sans revalider la propriété et la disponibilité, un attaquant peut substituer un SKU premium au prix de base.

## Théorie

Traitez les paramètres métier comme de simples indications : récupérez toujours à nouveau les enregistrements faisant autorité par leur ID côté serveur avant la validation.

## Pratique

- Finalisez l'achat d'un article bon marché ; interceptez le POST final et échangez les identifiants pour un SKU coûteux dans un catalogue de laboratoire.

## Outils

- **Burp Suite**

## Références

- PortSwigger Web Security Academy : Business logic vulnerabilities
- OWASP WSTG : Testing for Business Logic
