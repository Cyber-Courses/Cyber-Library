---
title: "API Gateway"
order: 3
description: "Attaquer API Gateway : exposition par la resource policy, contournement d'authorizer et rôle d'intégration sous lequel s'exécutent les appels backend."
keywords:
  - API Gateway
  - resource policy
  - authorizer
  - integration role
  - REST API
---

# API Gateway

API Gateway place les fonctions Lambda et d'autres services AWS derrière une API HTTP, et trois éléments y sont pertinents pour un attaquant : la **resource policy** qui décide qui peut l'invoquer, l'**authorizer** censé verrouiller les routes protégées, et le **rôle d'intégration** sous lequel s'exécutent les appels backend. Une resource policy laxiste expose une API privée, un authorizer défaillant se contourne pour atteindre des routes protégées, et un rôle d'intégration puissant transforme un endpoint accessible en actions contre d'autres services.

## Ce qui se rattache ici

- **[Resource policy](resource-policy.md)** : une politique permissive qui expose une API privée ou interne.
- **[Contournement d'authorizer](authorizer-bypass.md)** : vaincre un authorizer Lambda ou JWT pour atteindre des routes protégées.
- **[Rôle d'intégration](integration-role.md)** : le rôle de crédentiels sous lequel s'exécutent les appels AWS proxifiés.

## Énumérer les API

```bash
aws apigateway get-rest-apis
aws apigatewayv2 get-apis
aws apigateway get-resources --rest-api-id <id>
aws apigateway get-method --rest-api-id <id> --resource-id <rid> --http-method GET
# steal usable API keys, values included
aws apigateway get-api-keys --include-values
```

Une clé renvoyée avec sa valeur s'utilise directement comme en-tête `x-api-key` contre le stage, si bien que `get-api-keys --include-values` transforme un accès en lecture en appels fonctionnels.

## Références

- [HackTricks Cloud : AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS : resource policies d'API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-resource-policies.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Conférences fwd:cloudsec](https://fwdcloudsec.org/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
