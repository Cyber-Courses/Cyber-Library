---
title: "Contrôle d'accès au niveau des endpoints : routes, surfaces d'administration, GraphQL et fonctions cachées"
description: Failles d'autorisation où une route HTTP, une méthode RPC ou une fonction interne est appelable sans le bon rôle, la bonne portée ou le bon contrôle de tenant.
keywords:
  - autorisation au niveau fonction
  - autorisation d'endpoint
  - contrôle de rôle manquant
  - élévation de privilèges horizontale
  - élévation de privilèges verticale
---

# Niveau endpoint

L'autorisation au **niveau fonction** (endpoint) régit *quelles* opérations un appelant peut utiliser : API d'administration, outils internes exposés via HTTP, routes de debug, et champs ou resolvers GraphQL. L'application peut authentifier l'utilisateur et rester fautive si l'**autorisation** de l'**action** est manquante, incohérente, ou appliquée uniquement côté client.

## Pages

- [Contrôle de rôle manquant](missing-role-check.md)
- [Introspection GraphQL](graphql-introspection.md)
- [Swagger UI et surface OpenAPI](swagger-ui.md)
- [Contournement par method override HTTP](method-override-bypass.md)
- [Exposition d'endpoint de debug](debug-endpoint-exposure.md)
- [Panneaux d'administration](admin-panels.md)
- [Ressources de test en production](test-resources.md)
- [Fonction d'administration cachée](hidden-admin-function.md)
