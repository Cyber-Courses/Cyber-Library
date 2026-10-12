---
title: "Appartenance à un groupe : s'ajouter à un groupe plus privilégié"
order: 3
description: "iam:AddUserToGroup pour rejoindre un groupe qui porte plus d'autorisations que votre principal actuel."
keywords:
  - AddUserToGroup
  - appartenance à un groupe
  - escalade de privilèges
  - IAM
  - groupe
---

# Appartenance à un groupe

Un utilisateur hérite de toutes les politiques attachées aux groupes auxquels il appartient. `iam:AddUserToGroup` vous permet d'ajouter votre utilisateur à n'importe quel groupe, de sorte que rejoindre un groupe plus privilégié (un groupe d'administrateurs, ou un groupe doté d'une large politique attachée) vous promeut en un seul appel.

## Rejoindre un groupe privilégié

```bash
aws iam list-groups                       # find a group with strong policies
aws iam list-attached-group-policies --group-name <group>
aws iam add-user-to-group --group-name <group> --user-name <you>
```

## Notes d'exploitation

- Les autorisations héritées s'appliquent immédiatement aux nouvelles sessions ; rafraîchissez les identifiants si une session en cache est antérieure au changement.
- Cela diffère d'[AttachGroupPolicy](policy-manipulation/attach-group-policy.md) : ici le groupe est déjà privilégié et vous vous contentez de le rejoindre.

## Outils

- **AWS CLI** (`iam add-user-to-group`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs: AWS privilege escalation (AddUserToGroup)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
