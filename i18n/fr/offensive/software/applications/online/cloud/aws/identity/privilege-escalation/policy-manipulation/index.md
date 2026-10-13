---
title: "Manipulation de politiques"
order: 1
description: "S'accorder des permissions en modifiant la politique IAM : nouvelles versions de politique par défaut et attachement ou intégration en ligne de politiques utilisateur, groupe et rôle."
keywords:
  - policy manipulation
  - CreatePolicyVersion
  - AttachUserPolicy
  - PutUserPolicy
  - inline policy
  - managed policy
---

# Manipulation de politiques

Quand un principal peut écrire de la politique IAM, il peut s'écrire lui-même en administrateur. Ce sont les escalades les plus directes d'AWS : une seule écriture `iam:`, aucune nouvelle ressource, aucun hôte. Elles se répartissent en trois formes : changer quelle **version** d'une politique managée est active, **attacher** une politique managée et **intégrer en ligne** un document de politique.

## Versions de politiques managées

- **[CreatePolicyVersion](create-policy-version.md)** : publier une nouvelle version par défaut d'une politique que vous pouvez modifier, accordant un accès complet.
- **[SetDefaultPolicyVersion](set-default-policy-version.md)** : activer une version existante plus permissive.

## Attacher des politiques managées

- **[AttachUserPolicy](attach-user-policy.md)** : attacher `AdministratorAccess` à un utilisateur que vous contrôlez.
- **[AttachGroupPolicy](attach-group-policy.md)** : attacher une politique privilégiée à un groupe auquel vous appartenez.
- **[AttachRolePolicy](attach-role-policy.md)** : attacher une politique privilégiée à un rôle que vous pouvez assumer.

## Intégrer en ligne des documents de politique

- **[PutUserPolicy](put-user-policy.md)** : intégrer en ligne un document autorisant tout sur un utilisateur.
- **[PutGroupPolicy](put-group-policy.md)** : intégrer en ligne un document autorisant tout sur un groupe.
- **[PutRolePolicy](put-role-policy.md)** : intégrer en ligne un document autorisant tout sur un rôle.

## Références

- [Rhino Security Labs : élévation de privilèges AWS (voies de politique)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox : iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud : AWS](https://hackingthe.cloud/)
