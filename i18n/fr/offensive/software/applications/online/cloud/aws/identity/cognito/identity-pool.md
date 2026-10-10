---
title: "Identity pool : des identifiants pour une identité non authentifiée"
order: 1
description: "Échanger une identité Cognito non authentifiée ou invitée contre les identifiants du rôle IAM de l'identity pool."
keywords:
  - Cognito
  - identity pool
  - non authentifié
  - invité
  - rôle IAM
---

# Identity pool

Un identity pool Cognito mappe les utilisateurs de l'application (et, lorsque c'est activé, les **invités non authentifiés**) à des rôles IAM et renvoie de vrais identifiants AWS. Si le pool autorise les identités non authentifiées, quiconque possède l'identifiant du pool obtient les identifiants du rôle invité, et ces rôles sont fréquemment sur-privilégiés.

## Obtenir des identifiants invités

```bash
# the pool id is often visible in the app's JS or mobile bundle
aws cognito-identity get-id --identity-pool-id <region>:<pool-id>
aws cognito-identity get-credentials-for-identity --identity-id <id>
# export the returned keys and enumerate what the guest role can do
```

## Notes d'exploitation

- Les identifiants de pool sont des valeurs côté client, traitez donc tout front end utilisant Cognito comme une source pour les obtenir.
- La portée du rôle invité est le prix : énumérez-la avec [identity enumeration](../enumeration.md), car ces rôles accordent souvent un accès S3 ou DynamoDB bien au-delà de ce dont l'application a besoin.
- Les identifiants d'identité authentifiée suivent les mêmes appels une fois que vous détenez un jeton de user pool, ce qui renvoie à [token claims](token-claims.md).

## Outils

- **AWS CLI** (`cognito-identity get-id` / `get-credentials-for-identity`).
- **Pacu** : modules d'énumération Cognito.

## Références

- [HackTricks Cloud: AWS Cognito identity pools](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [AWS: Cognito identity pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-identity.html)
- [cognito-scanner](https://pypi.org/project/cognito-scanner/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
