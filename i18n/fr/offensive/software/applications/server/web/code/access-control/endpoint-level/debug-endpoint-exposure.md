---
title: "Endpoints de debug et d'actuator : stack traces, dumps d'environnement et listes de routes en production"
description: Routes HTTP de développement ou de diagnostic encore enregistrées dans les builds de production, renvoyant souvent des stack traces ou des actions du plan de contrôle.
keywords:
  - route de debug
  - endpoint de test
  - actuator
  - API réservée au développement
---

# Endpoints de debug

## Contexte

Les routes de type debug et actuator sont souvent livrées avec un framework et censées être désactivées ou restreintes au réseau en production. Lorsqu'elles restent liées et atteignables, elles renvoient des détails d'environnement, de la configuration ou des actions en un clic (vidage de cache, bascule de fonctionnalité) que les utilisateurs normaux ne devraient pas pouvoir déclencher.

## Théorie

Le fingerprinting s'appuie sur des chemins prévisibles (`/actuator`, `/debug`, valeurs par défaut du framework), des corps d'erreur distincts selon `GET` ou `POST`, et des chaînes de bannière dans les enveloppes d'erreur JSON. L'impact est de l'information et parfois un changement d'état, non une nouvelle primitive HTTP.

## Pratique

### Découverte de chemins et de verbes dans le périmètre

- Dans un environnement de test, faites un `GET` et un `POST` sur des chemins par défaut courants avec une petite wordlist dérivée de la documentation de la stack. Notez les codes de statut et les types de contenu ; un `200` avec un objet JSON `beans` ou `env` signale une exposition d'actuator classique quand le produit est de type Java/Spring.

## Outils

- **ffuf**
- **curl**
- **Burp Suite**
