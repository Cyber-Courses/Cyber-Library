---
title: "Étape de workflow client vs version de la machine à états serveur : sauter et bondir dans les flux multi-étapes"
description: "Indicateurs d'étape côté client qui se désynchronisent d'une version de workflow ou d'une machine à états côté serveur."
keywords:
  - workflow
  - machine à états
---

# Surface vs version

## Contexte

La SPA affiche « Étape 3 sur 5 » alors que le moteur de workflow de l'API est sur la version 2 avec des transitions autorisées différentes. Des POST directs vers `/step/4` peuvent réussir si le serveur ne vérifie que le rôle de session, et non le currentState issu de la table d'états.

## Théorie

Les conceptions robustes exposent un jeton de workflow opaque ou une URL d'étape générée par le serveur après chaque transition.

## Pratique

- Complétez normalement les étapes 1 et 2 ; puis envoyez le payload de l'étape 4 par POST alors que l'interface affiche encore l'étape 3, dans un tenant de test.

## Outils

- **Burp Suite**

## Références

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
