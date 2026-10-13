---
title: "Endpoints de test et de staging en production : données d'exemple, paiements factices et routes de debug"
description: Routes de staging, comptes de test et données d'exemple laissés dans les déploiements en production, souvent avec des règles d'accès plus faibles que les fonctionnalités de production.
keywords:
  - comptes de test
  - données de staging
  - identifiants par défaut
  - mode démo
---

# Ressources de test

## Contexte

Les fonctionnalités « test » et « démo » sont souvent implémentées avec des limites de débit assouplies, des mots de passe connus, ou des champs de debug supplémentaires. Elles peuvent partager le même chemin de code que les fonctionnalités de production via un feature flag, ce qui les rend très rentables dans le périmètre lorsque le flag est mal positionné en prod.

## Théorie

La découverte provient de chaînes statiques dans le JS client, de `robots.txt`, de noms DNS (`staging.example.com` pointant vers une infra semblable à la prod), et de messages d'erreur qui nomment des tenants de test. L'abus relève généralement de la même classe BOLA ou au niveau fonction avec une surface de garde plus petite.

## Pratique

### Rechercher les hostnames et flags de test dans les artefacts client

- Extrayez des chaînes des bundles mobiles et web pour les sous-domaines `test`, `demo`, `internal`, puis essayez ces hostnames et préfixes de chemin sur le programme dans le périmètre avec les mêmes comptes de test que vous avez déjà.

## Outils

- **strings**
- **Burp Suite**
- **curl**
