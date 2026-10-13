---
title: "SetDefaultPolicyVersion : activer une version existante plus permissive"
description: "iam:SetDefaultPolicyVersion pour activer une version existante mais plus permissive d'une politique managée."
keywords:
  - SetDefaultPolicyVersion
  - managed policy
  - policy version
  - rollback
  - admin
---

# SetDefaultPolicyVersion

Une politique managée peut déjà porter une version antérieure, plus large, qui n'est plus la version par défaut. `iam:SetDefaultPolicyVersion` active n'importe quelle version stockée : si une politique attachée à vous possède une version non par défaut permissive dans son historique, vous basculez dessus sans écrire de nouveau document.

## Lister les versions et basculer

```bash
aws iam list-policy-versions --policy-arn arn:aws:iam::<acct>:policy/<policy>
# find a version whose document is broader than the current default
aws iam set-default-policy-version \
  --policy-arn arn:aws:iam::<acct>:policy/<policy> --version-id v2
```

## Notes d'exploitation

- Ceci ne requiert que `iam:SetDefaultPolicyVersion`, une permission plus étroite que `CreatePolicyVersion`, et ne laisse aucun nouveau document de politique derrière.
- Inspectez le document de chaque version stockée avec `get-policy-version` pour trouver la plus permissive avant de basculer.

## Outils

- **AWS CLI** (`iam list-policy-versions` / `set-default-policy-version`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs : élévation de privilèges AWS (SetDefaultPolicyVersion)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
