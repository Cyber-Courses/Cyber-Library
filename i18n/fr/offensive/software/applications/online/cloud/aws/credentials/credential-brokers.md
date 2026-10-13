---
order: 5
title: "Courtiers de credentials : services qui émettent des credentials pour d'autres principaux"
description: "Abuser des services qui servent d'intermédiaires pour des credentials vers d'autres principaux, transformant une permission étroite en nouveaux credentials de rôle ou d'utilisateur."
keywords:
  - courtier de credentials
  - credentials de rôle
  - jeton
  - STS
  - escalade de privilèges
---

# Courtiers de credentials

Plusieurs actions AWS existent pour distribuer des credentials, et détenir l'une d'elles transforme une permission étroite en un nouveau jeu de clés pour un autre principal. Ce sont les pendants « émetteurs de credentials » des chemins d'[escalade de privilèges](../identity/privilege-escalation/index.md) : là où ceux-ci changent ce qu'un principal peut faire, ceux-là produisent directement des credentials utilisables.

## Les actions de courtage

```bash
# mint a second long-term key for a user you can write to
aws iam create-access-key --user-name <victim>

# a role's credentials via assume, where the trust allows you
aws sts assume-role --role-arn <arn> --role-session-name s

# EC2 Instance Connect pushes a key you then use to reach the box and its role
aws ec2-instance-connect send-ssh-public-key \
  --instance-id <id> --instance-os-user ec2-user --ssh-public-key file://key.pub
```

## Jetons de session vers une console

`sts:GetFederationToken` et `sts:GetSessionToken` émettent un triplet de credentials qui se convertit en URL de connexion navigateur, un point d'ancrage durable et souvent moins surveillé que la CLI :

```bash
aws_consoler -a ASIA... -s <secret> -t <session-token>   # -> console sign-in URL
```

Les sessions `GetFederationToken` durent jusqu'à 36 heures et continuent de fonctionner après la rotation de la clé source, ce qui fait de l'URL un artefact de persistance discret ; voir [jetons STS](sts-tokens.md).

## Notes d'exploitation

- `iam:CreateAccessKey` sur un autre utilisateur est le courtier le plus discret : la nouvelle clé est valide immédiatement et survit à la fin de la session de la victime, couvert sous [création de credentials](../identity/privilege-escalation/credential-creation/index.md).
- `sts:AssumeRole` et les variantes web-identity et SAML sont des courtiers cadrés par la politique de confiance ; voir [assomption de rôle](../identity/role-assumption/index.md) et [fédération](../identity/federation/index.md).
- Les courtiers liés à un service (les pools d'identités Cognito distribuant des credentials de rôle non authentifiés) vivent sous leur service dans [identité](../identity/cognito/index.md).

## Outils

- **AWS CLI** (`iam create-access-key`, `sts assume-role`, `sts get-federation-token`, `ec2-instance-connect`).
- **aws_consoler** (NetSPI) : transformer un triplet de credentials obtenu par courtage en URL de connexion à la console.
- **Pacu** (`iam__backdoor_users_keys`) : enchaîne les actions de courtage découvertes par le scan de privesc.

## Références

- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS credential abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [aws_consoler (NetSPI)](https://github.com/NetSPI/aws_consoler)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
