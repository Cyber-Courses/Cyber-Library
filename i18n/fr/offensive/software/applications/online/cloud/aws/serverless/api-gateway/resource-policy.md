---
title: "Resource policy : invoquer une API privée via une politique permissive"
order: 1
description: "Abuser d'une resource policy d'API Gateway permissive pour invoquer des API privées ou internes."
keywords:
  - API Gateway
  - resource policy
  - private API
  - invoke
  - access
---

# Resource policy

Une **resource policy** d'API Gateway décide quels principals, VPC sources ou plages d'IP peuvent invoquer l'API. Lorsqu'elle est écrite avec un principal en wildcard, une condition manquante ou une source trop large, une API censée être privée devient invocable par quiconque peut atteindre son endpoint, y compris des API strictement internes exposées à un attaquant qui détient n'importe quel crédentiel de compte ou position réseau.

## Lire la politique et invoquer

```bash
aws apigateway get-rest-api --rest-api-id <id> --query 'policy'
# invoke the stage endpoint directly
curl -s "https://<id>.execute-api.<region>.amazonaws.com/<stage>/<path>"
```

Pour une API `PRIVATE` liée à un endpoint de VPC, invoquez-la depuis l'intérieur du VPC (ou via une SSRF depuis une ressource qui s'y trouve) lorsque la politique ne restreint pas davantage l'appelant.

## Notes d'exploitation

- Un `Principal: "*"` sans `Condition` rend l'API invocable par le monde entier à son endpoint ; une condition `aws:SourceVpce` n'aide que l'attaquant déjà présent dans ce VPC.
- Les API privées s'atteignent depuis une ressource compromise située dans le VPC ou via une primitive SSRF : associez donc ceci à la [SSRF](../../credentials/instance-metadata/ssrf.md).
- Cartographiez d'abord les routes avec `get-resources` : les routes non documentées derrière la gateway sont souvent les plus intéressantes.

## Outils

- **AWS CLI** (`apigateway get-rest-api`, `get-resources`) : découverte de la politique et des routes.
- **curl** / **Burp Suite** : invoquer et fuzzer les routes exposées.

## Références

- [AWS : exemples de resource policy d'API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-resource-policies-examples.html)
- [HackTricks Cloud : AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Conférences fwd:cloudsec](https://fwdcloudsec.org/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
