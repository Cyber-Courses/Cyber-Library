---
title: "Rôle d'exécution : voler les identifiants d'une Lambda depuis le runtime"
order: 1
description: "Voler les identifiants du rôle d'exécution d'une fonction Lambda depuis les variables d'environnement du runtime et le point de terminaison de métadonnées."
keywords:
  - Lambda
  - rôle d'exécution
  - variables d'environnement
  - identifiants
  - AWS_SESSION_TOKEN
---

# Rôle d'exécution

Chaque fonction Lambda assume un **rôle d'exécution**, et le runtime expose les identifiants temporaires de ce rôle au processus de la fonction sous forme de variables d'environnement. Toute exécution de code au sein de la fonction, que ce soit via un [remplacement de code](code-and-layers.md), une dépendance ou une injection applicative, peut les lire et continuer en tant que ce rôle. Par ailleurs, `lambda:GetFunctionConfiguration` expose les propres variables d'environnement de la fonction depuis l'API, lesquelles contiennent fréquemment des mots de passe de bases de données et des clés d'API.

## Lire les identifiants depuis l'intérieur de la fonction

```bash
# the execution role's temporary credentials are injected as env vars
env | grep -E 'AWS_ACCESS_KEY_ID|AWS_SECRET_ACCESS_KEY|AWS_SESSION_TOKEN|AWS_REGION'
# AWS_LAMBDA_RUNTIME_API serves the same credentials to the runtime
```

Exportez ces trois valeurs en local et vous devenez le rôle d'exécution.

## Lire la configuration et les secrets depuis l'API

```bash
aws lambda get-function-configuration --function-name <fn> \
  --query 'Environment.Variables'
aws lambda list-functions \
  --query 'Functions[].[FunctionName,Role,Environment.Variables]'
```

## Notes d'exploitation

- Les identifiants injectés sont ceux du rôle d'exécution ; leur pouvoir correspond exactement à ce que détient ce rôle, il faut donc l'énumérer avec les clés dérobées.
- `get-function-configuration` renvoie les variables d'environnement en clair, sauf si elles ont été chiffrées avec une clé KMS client que vous ne pouvez pas utiliser ; les secrets non chiffrés ressortent directement.
- Pour obtenir une exécution de code là où aucune injection n'est possible, remplacez la fonction via [le code et les couches](code-and-layers.md).

## Outils

- **AWS CLI** (`lambda get-function-configuration`, `list-functions`) : énumération de la configuration et des secrets.
- **Pacu** (`lambda__enum`) : énumération en masse des fonctions et de leurs variables d'environnement.

## Références

- [HackTricks Cloud : AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS : variables d'environnement du runtime Lambda](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
- [Rhino Security Labs : élévation de privilèges AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team : techniques AWS](https://stratus-red-team.cloud/attack-techniques/AWS/)
