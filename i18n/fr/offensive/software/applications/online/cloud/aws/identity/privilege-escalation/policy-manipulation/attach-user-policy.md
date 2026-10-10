---
title: "AttachUserPolicy : attacher AdministratorAccess à votre utilisateur"
description: "iam:AttachUserPolicy pour attacher AdministratorAccess ou une autre politique managée privilégiée à un utilisateur que vous contrôlez."
keywords:
  - AttachUserPolicy
  - managed policy
  - AdministratorAccess
  - user
  - attach
---

# AttachUserPolicy

`iam:AttachUserPolicy` attache une politique managée à un utilisateur. Si vous la détenez pour votre propre utilisateur, attachez la politique managée par AWS `AdministratorAccess` et vous êtes administrateur en un seul appel.

## Attacher les droits admin

```bash
aws iam attach-user-policy --user-name <you> \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

## Notes d'exploitation

- `AdministratorAccess` est une politique managée par AWS présente dans chaque compte, aucune création de politique n'est donc nécessaire.
- Fonctionne uniquement contre les utilisateurs ; les équivalents pour les rôles et les groupes sont [AttachRolePolicy](attach-role-policy.md) et [AttachGroupPolicy](attach-group-policy.md).

## Outils

- **AWS CLI** (`iam attach-user-policy`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs : élévation de privilèges AWS (AttachUserPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
