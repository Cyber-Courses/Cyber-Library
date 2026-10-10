---
title: "Identity Store : ajout d'utilisateurs et appartenance à des groupes"
order: 2
description: "Abuser du magasin d'identités d'Identity Center pour ajouter ou modifier des utilisateurs et des appartenances de groupe, et hériter de leurs accès."
keywords:
  - Identity Store
  - IAM Identity Center
  - utilisateur
  - appartenance à un groupe
  - SSO
---

# Identity Store

Le magasin d'identités contient les utilisateurs et groupes d'Identity Center. Comme les affectations de jeux d'autorisations ciblent des utilisateurs et des groupes, un accès en écriture au magasin vous permet de vous ajouter à un groupe qui porte déjà des affectations, ou de créer un utilisateur associé à un accès privilégié.

## Rejoindre un groupe privilégié

```bash
aws identitystore list-groups --identity-store-id <store-id>
aws identitystore create-group-membership --identity-store-id <store-id> \
  --group-id <privileged-group-id> \
  --member-id UserId=<your-user-id>
```

## Notes d'exploitation

- Ciblez les groupes qui détiennent déjà des affectations de compte : l'appartenance en hérite les jeux d'autorisations sans toucher aux affectations elles-mêmes.
- Lorsque Identity Center se synchronise depuis un IdP externe (SCIM), l'annuaire externe est le véritable point de contrôle ; les modifications peuvent être écrasées à la prochaine synchronisation, ce qui rend cette approche la plus solide sur un magasin géré en interne.
- À combiner avec le [jeu d'autorisations](permission-set.md) : le magasin décide du qui, le jeu d'autorisations décide du quoi.

## Outils

- **AWS CLI** (commandes `identitystore`).

## Références

- [HackTricks Cloud: AWS Identity Center](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-iam-and-sts-enum.html)
- [AWS: Identity Store](https://docs.aws.amazon.com/singlesignon/latest/IdentityStoreAPIReference/welcome.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
