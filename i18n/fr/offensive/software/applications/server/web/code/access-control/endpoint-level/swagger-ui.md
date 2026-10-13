---
title: "Swagger UI et documentation OpenAPI exposés en production : risque d'exploration interactive de l'API"
description: Documents OpenAPI JSON et interfaces de documentation interactives exposés qui énumèrent les routes, les paramètres et les schémas d'authentification d'un service.
keywords:
  - Swagger UI
  - OpenAPI
  - documentation d'API
  - openapi.json
  - console try it
---

# Swagger UI

## Contexte

Un endpoint `/swagger-ui`, `/openapi.json` ou similaire atteignable publie le contrat que le serveur pense implémenter. La valeur offensive est une couverture complète des chemins et des paramètres pour planifier les tests BOLA, d'injection et au niveau fonction, incluant souvent des opérations dépréciées ou taguées internes.

## Théorie

Les spécifications écrites à la main peuvent diverger du code ; les spécifications générées par codegen suivent le code de plus près. Dans les deux cas, le fichier est un artefact de reconnaissance. Les navigateurs avec une session active peuvent émettre des requêtes « try it » depuis la même origine, ce qui importe pour la façon dont vous enchaînez avec d'autres problèmes en labo ; traitez l'endpoint de documentation comme faisant partie du même modèle de site que l'API pour les expériences de session et de CSRF lorsque celles-ci sont dans le périmètre.

## Pratique

### Récupérer la spécification lisible par machine

- Faites un `curl -sS` sur l'URL `openapi.json` ou `swagger.json` dans le périmètre, ou téléchargez depuis l'onglet réseau de l'interface. Analysez les chemins et les méthodes en une matrice pour les tests de rôle.

### Tri par tag et par version

- Lisez les blocs `tags`, `x-internal`, et les chemins `/v1` vs `/v2`. Les blocs de versions plus anciennes conservent souvent des handlers plus faibles actifs.

## Outils

- **curl**
- **jq**
- **Burp Suite**
