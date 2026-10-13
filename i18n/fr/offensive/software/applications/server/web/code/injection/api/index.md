---
order: 9
title: "Injection d'API : abuser du dispatch de requêtes structurées"
description: "Le code serveur qui analyse et dispatche les requêtes d'API structurées décide ce qui s'exécute, pour qui et contre quel backend. Lorsqu'une entrée non fiable façonne ce dispatch, le contrat lui-même devient la surface d'attaque, organisé ici par style de dispatch."
keywords:
  - injection d'API
  - autorisation d'API
  - dispatch RPC
  - GraphQL
  - SOAP
  - gRPC
  - JSON-RPC
---

# API

Un endpoint d'API est un dispatcheur. Il prend une requête structurée, une requête de graphe, une enveloppe RPC, un document SOAP, un message protobuf, décide quel handler ou resolver s'exécute, lie les champs de la requête aux arguments de ce handler, et appelle souvent d'autres backends pour le compte de l'appelant. Chacune de ces étapes fait confiance à la structure de la requête, et lorsqu'un attaquant contrôle cette structure, la logique de dispatch elle-même est la vulnérabilité, distincte de toute injection dans les données qu'elle transporte.

Les failles ici sont rarement une seule chaîne contaminée atteignant un sink. Ce sont des décalages : entre l'opération que l'appelant a nommée et celle que le serveur exécute, entre l'endroit où l'autorisation est vérifiée et celui où le travail sensible se produit, entre la forme que le validateur attendait et la forme que le dispatcheur accepte. Le résultat est l'accès à des méthodes, des champs ou des objets que le contrat était censé tenir hors de portée.

## Organisé par style de dispatch

Chaque protocole analyse et route les requêtes différemment, de sorte que les schémas d'abus se regroupent par style :

- **[GraphQL](graphql/index.md)** : un seul endpoint exécutant un arbre de requête façonné par le client contre des resolvers par champ. La surface est l'autorisation au niveau des champs, le transfert d'arguments vers les backends, et le coût de l'amplification par alias et par groupage.
- **[gRPC](grpc/index.md)** : des messages protobuf dispatchés vers des méthodes de service. La surface est la réflexion de serveur, les métadonnées de confiance et le typage dynamique de messages (`Any`, `oneof`).
- **[SOAP](soap/index.md)** : des enveloppes XML routées par action et par corps. La surface est la confusion de dispatch action contre corps, le traitement des en-têtes (WS-Security, WS-Addressing) et le unmarshaling des paramètres du corps.
- **[JSON-RPC](json-rpc/index.md)** : un nom de méthode et des params mappés sur une procédure serveur. La surface est la résolution du nom de procédure, la gestion de la forme des paramètres et les tableaux de groupage.

## Le thème récurrent

À travers les quatre, les attaques les plus fortes exploitent une frontière que le framework a rendue invisible : l'autorisation appliquée à la frontière HTTP mais pas au resolver, un dispatch calé sur un champ tandis que l'exécution en lit un autre, un validateur qui a vu une structure différente de celle que le handler a finalement consommée. Chaque page relie l'abus au code du handler ou du resolver et à la frontière de validation qu'il franchit.

## Outils

- **Burp Suite** : interception et rejeu de requêtes d'API structurées à travers les protocoles.
- **InQL** : introspection de schéma GraphQL et génération de requêtes dans Burp.
- **grpcurl** : énumération et appel de services gRPC.
- **Postman** : composition de requêtes de style SOAP, JSON-RPC et REST pour tester le dispatch.

## Références

- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
- [OWASP Web Security Testing Guide: API Testing](https://owasp.org/www-project-web-security-testing-guide/)
