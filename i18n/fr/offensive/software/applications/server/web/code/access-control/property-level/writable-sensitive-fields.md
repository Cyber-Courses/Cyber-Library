---
title: "Champs de rôle, de propriétaire et de facturation modifiables : élévation de privilèges via des propriétés d'API modifiables"
description: Une famille ciblée de mass assignment où des colonnes de rôle, de propriété ou liées à l'argent changent via des API de mise à jour génériques.
keywords:
  - escalade de rôle
  - altération de l'owner id
  - altération de paiement
  - mass assignment
---

# Champs sensibles modifiables

## Contexte

`role`, `ownerId` / `userId` et les champs de facturation (`amount`, `price`, `plan`, `discount`) ont un fort impact lorsqu'ils sont modifiables par une route de mise à jour exposée à l'utilisateur. Le schéma reste de l'overposting ; le nom de la page met en avant l'impact métier et la priorité d'audit lors du triage.

## Théorie

Les chemins de code d'e-commerce et d'abonnement utilisent souvent un seul gestionnaire « mettre à jour la commande » ou « mettre à jour l'abonnement » avec un gros objet lié. Les champs financiers et la propriété sont parfois des clés optionnelles dans le même DTO que le client utilise pour des modifications inoffensives.

## Pratique

### Sondes à champ unique sur un objet panier ou abonnement en laboratoire

- `PATCH` avec seulement `{"total":1}` ou `{"ownerId":<other_user_id>}` contre un objet que la session ne devrait contrôler que partiellement. Observez le `200` et une lecture de suivi. Chaînez avec un BOLA sur `id` lorsque la route ne fixe pas l'objet non plus.

## Outils

- **Burp Suite**
- **curl**
