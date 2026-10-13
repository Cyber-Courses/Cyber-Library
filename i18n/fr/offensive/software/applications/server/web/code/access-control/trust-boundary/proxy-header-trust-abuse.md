---
title: "Abus de confiance des en-têtes de proxy : X-Forwarded-For, X-Real-IP et adresses client forgées"
description: Des backends qui font confiance à X-Forwarded-For, X-User-Id ou similaires venant d'un client capable d'atteindre l'application sans véritable forwarder de confiance.
keywords:
  - X-Forwarded-For
  - X-User-Id
  - proxy de confiance
  - usurpation d'en-tête
---

# Abus des en-têtes de proxy

## Contexte

Les reverse proxies définissent `Forwarded`, `X-Forwarded-For` et parfois des en-têtes d'identité propres à l'application. Si le conteneur applicatif est exposé à Internet et fait malgré tout confiance à ces en-têtes, un client peut définir `X-User-Id: 1` ou un `X-Forwarded-For` en loopback pour influencer **qui l'application pense être l'appelant** ou quelle IP l'application journalise pour les décisions de **rate**.

## Théorie

Le modèle de confiance exige un premier saut qui retire ou écrase les en-têtes de forward fournis par le client. Lorsque ce saut est absent, l'usage par l'application de `req.ip` ou `req.userId` issus des en-têtes est contrôlé par l'attaquant. Des schémas similaires s'appliquent à `X-Original-URL` sur certaines stacks lorsqu'il est utilisé pour des décisions de routage dans des configurations erronées.

## Pratique

### Envoyer des en-têtes d'identité sur une requête directe vers l'application en laboratoire

- Dans un déploiement de test qui reproduit une mauvaise disposition, interrogez (`curl`) le port de l'application avec `X-User-Id` ou `X-Role` et une requête par ailleurs non authentifiée ou à faible session. Observez si le comportement change par rapport à une requête sans ces en-têtes.

## Outils

- **curl**
- **Burp Suite**
