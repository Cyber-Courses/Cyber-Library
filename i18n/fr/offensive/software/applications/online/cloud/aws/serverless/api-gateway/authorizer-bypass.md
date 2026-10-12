---
title: "Contournement d'authorizer : vaincre un authorizer Lambda ou JWT"
order: 2
description: "Contourner un authorizer Lambda ou JWT via le cache, la gestion des en-têtes ou des failles de logique pour atteindre des routes protégées."
keywords:
  - authorizer
  - JWT
  - Lambda authorizer
  - bypass
  - API Gateway
---

# Contournement d'authorizer

Un **authorizer** personnalisé (une fonction Lambda qui renvoie une politique IAM, ou un authorizer JWT) est censé verrouiller les routes protégées. Trois faiblesses récurrentes le mettent en échec : la **mise en cache** du résultat d'autorisation indexée sur une source d'identité contrôlée par l'attaquant, une **gestion laxiste des tokens ou des en-têtes** dans un authorizer Lambda, et des failles de **portée de politique** où la politique renvoyée autorise plus que la méthode demandée.

## Sonder l'authorizer

```bash
aws apigateway get-authorizers --rest-api-id <id>
# how the identity source is keyed drives the cache attack
aws apigateway get-authorizer --rest-api-id <id> --authorizer-id <aid> \
  --query '[identitySource,authorizerResultTtlInSeconds,type]'
```

## Contournements courants

```
# cache poisoning: if the result is cached per token value, a once-valid
# token is replayed within the TTL even after the session should be dead
Authorization: <previously-valid-token>

# Lambda authorizer that greps for a substring or trusts an unverified header
X-Forwarded-For, X-Original-Url, or a crafted token the handler parses loosely

# wildcarded returned policy: methodArn "*/*" lets one authorized route reach all
```

## Notes d'exploitation

- Lorsque `authorizerResultTtlInSeconds` est élevé et que la source d'identité est le token lui-même, un token fuité ou rejoué reste valable pendant toute la durée du TTL.
- Les authorizers Lambda qui construisent la politique d'autorisation avec une `Resource` en wildcard accordent toutes les routes dès qu'une seule route autorise.
- Les authorizers JWT qui ne verrouillent pas `aud`/`iss`, ou qui acceptent `alg: none`, sont vulnérables de la même façon que n'importe quel vérificateur JWT : voir les pages JWT côté web.

## Outils

- **AWS CLI** (`apigateway get-authorizers`) : lire la configuration de l'authorizer.
- **Burp Suite** / **jwt_tool** : forger et rejouer tokens et en-têtes.

## Références

- [AWS : authorizers Lambda d'API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)
- [HackTricks Cloud : AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Conférences fwd:cloudsec](https://fwdcloudsec.org/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
