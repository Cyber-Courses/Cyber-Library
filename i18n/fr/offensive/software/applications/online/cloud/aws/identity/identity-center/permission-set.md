---
title: "Jeu d'autorisations : s'accorder un rôle dans un compte cible"
order: 1
description: "Affecter ou modifier un jeu d'autorisations Identity Center pour s'accorder un rôle dans un compte cible."
keywords:
  - jeu d'autorisations
  - IAM Identity Center
  - affectation
  - compte
  - SSO
---

# Jeu d'autorisations

Un jeu d'autorisations est un modèle qu'Identity Center provisionne sous forme de rôle IAM dans un compte membre. Avec les autorisations d'administration SSO, vous affectez un jeu d'autorisations existant à fort privilège (ou en modifiez un) à votre propre utilisateur ou groupe dans un compte cible, puis vous vous connectez via le portail SSO en tant que ce rôle.

## Affecter un jeu d'autorisations

```bash
aws sso-admin list-permission-sets --instance-arn <instance-arn>
aws sso-admin create-account-assignment --instance-arn <instance-arn> \
  --permission-set-arn <ps-arn> \
  --principal-type USER --principal-id <your-identitystore-user-id> \
  --target-type AWS_ACCOUNT --target-id <account-id>
```

Récupérez ensuite les identifiants via le portail SSO ou `aws sso login`.

## Notes d'exploitation

- Modifier les politiques inline ou gérées d'un jeu d'autorisations propage le changement à tous les comptes où il est affecté, soit une escalade de grande ampleur.
- Le compte de gestion ou d'administration déléguée détient ces autorisations ; l'atteindre (par exemple via [Organizations](../organizations/member-account-role.md)) débloque tout le parc.
- Les affectations provisionnent de vrais rôles IAM, de sorte que l'accès accordé se comporte ensuite comme n'importe quel rôle endossé.

## Outils

- **AWS CLI** (commandes `sso-admin`, `aws sso login`).
- **Pacu** : énumération d'Identity Center.

## Références

- [HackTricks Cloud: AWS SSO and Identity Center](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-iam-and-sts-enum.html)
- [AWS: permission sets](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
