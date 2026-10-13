---
title: "Fonctions d'administration cachées : routes non divulguées, feature flags et actions réservées aux opérateurs"
description: Routes HTTP et méthodes RPC non documentées qui existent encore en production, souvent avec une revue plus faible ou une autorisation manquante.
keywords:
  - API non documentée
  - route interne
  - endpoint hérité
  - API de debug
---

# Fonctions d'administration cachées

## Contexte

Ce ne sont pas nécessairement des backdoors malveillantes. Ce sont souvent des crochets de support, des endpoints de migration ponctuels, ou des outils d'administration « temporaires » qui n'ont jamais quitté l'arborescence. Ils échappent à l'OpenAPI public et parfois au même niveau de revue que les routes de première classe.

## Théorie

La reconnaissance combine les lacunes de la spécification publique avec les chaînes de caractères des binaires mobiles, les traces d'erreur et les ensembles de champs GraphQL qui n'apparaissent pas dans le client commercialisé. Le mode de défaillance reste une autorisation de niveau fonction manquante ou faible sur un handler puissant.

## Pratique

### Croiser la surface mobile et web

- Comparez la liste des chemins ou des operationId de la liste codée en dur d'un binaire mobile aux appels réseau de l'application web. Les divergences signalent des routes serveur que seuls certains clients utilisent.

## Outils

- **jadx** / **apktool** (pour les artefacts Android de labo dans le périmètre)
- **Burp Suite**
- **strings**
