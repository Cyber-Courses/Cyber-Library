---
order: 4
title: "Contournement d'approbation dans les workflows : auto-approbation, liaison faible de l'approbateur, jetons périmés et lacunes de séparation des tâches"
description: "Contourner les étapes d'approbation humaine ou système : liaison faible de l'approbateur, auto-approbation, jetons périmés ou absence de séparation des tâches dans les API."
keywords:
  - contournement d'approbation
  - maker checker
  - workflow
---

# Contournement d'approbation

## Contexte

Le contournement d'approbation vise les transitions qui exigent un second principal (validation d'un manager, revue des risques). Les défaillances typiques incluent l'auto-approbation (même utilisateur comme demandeur et approbateur), les jetons d'approbation réutilisés, la liaison faible des identifiants d'approbation au payload de la requête, et des routes telles que `POST /approve` qui omettent les contrôles de rôle ou d'organisation.

## Théorie

Cartographiez les états (par exemple pending → approved → settled) et les identifiants autorisés à déclencher chaque transition. Recouvrement avec [Contrôle d'accès](../../../access-control/index.md) lorsque la défaillance est un rôle manquant sur une route ; cette page met l'accent sur les graphes de workflow qui comportent une transition d'approbateur explicite.

## Pratique

### Tentative d'auto-approbation

- Créez une requête en tant qu'utilisateur A, puis appelez l'endpoint d'approbation authentifié en tant que A lorsque le produit prétend assurer une séparation des tâches.

### Rejeu d'un identifiant d'approbation périmé

- Capturez `approvalId` depuis un flux terminé, démarrez une nouvelle requête et substituez l'ancien `approvalId` si le serveur l'accepte sans le lier à un nouveau hash de payload.

## Outils

- **Burp Suite**
- **curl**

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
