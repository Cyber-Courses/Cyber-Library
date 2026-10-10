---
title: "Cognito"
order: 5
description: "Abuser d'Amazon Cognito : assumption de rôle d'identity pool non authentifiée, auto-inscription et abus d'attributs de user pool, et revendications de jetons forgées ou élevées."
keywords:
  - Cognito
  - identity pool
  - user pool
  - revendications de jeton
  - non authentifié
  - auto-inscription
---

# Cognito

Amazon Cognito est le service d'identité grand public derrière de nombreuses applications web et mobiles, et il relie les utilisateurs d'application à de vrais identifiants IAM. Ce pont est la surface d'attaque : un identity pool mal configuré livre des identifiants AWS à des appelants anonymes, et un user pool ouvert laisse quiconque s'inscrire.

## Pages

- **[Identity pool](identity-pool.md)** : échanger une identité non authentifiée ou invitée contre les identifiants du rôle IAM du pool.
- **[User pool](user-pool.md)** : auto-inscription ouverte et attributs modifiables pour obtenir ou élever un accès.
- **[Token claims](token-claims.md)** : abuser des revendications des jetons d'ID et d'accès pour assumer une identité plus privilégiée.

La vérification non authentifiée la plus rapide est le flux d'identity pool : `aws cognito-identity get-id` pour un identifiant de pool, puis `get-credentials-for-identity`, qui renvoie des identifiants IAM lorsque le pool accorde un rôle non authentifié (voir [identity pool](identity-pool.md)).

## Outils

- **cognito-scanner** (`pip install cognito-scanner`) : création de compte non souhaitée, oracle d'existence de compte et escalade d'identifiants d'identity pool réunis en un seul scanner.
- **AWS CLI** (`cognito-identity`, `cognito-idp`) : les flux bruts `get-id` / `get-credentials-for-identity` et `sign-up` / `initiate-auth`.

## Références

- [HackTricks Cloud: AWS Cognito](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [cognito-scanner (PyPI)](https://pypi.org/project/cognito-scanner/)
- [Rhino Security Labs: Cognito research](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
