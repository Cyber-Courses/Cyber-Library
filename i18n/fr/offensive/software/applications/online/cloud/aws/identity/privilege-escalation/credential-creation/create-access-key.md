---
title: "CreateAccessKey : générer des clés pour un utilisateur plus privilégié"
description: "iam:CreateAccessKey pour générer un second jeu de clés à long terme pour un utilisateur plus privilégié."
keywords:
  - CreateAccessKey
  - clé d'accès
  - utilisateur
  - identifiants à long terme
  - IAM
---

# CreateAccessKey

`iam:CreateAccessKey` crée une clé d'accès à long terme pour un utilisateur. Exécutée contre un utilisateur plus privilégié, elle vous remet une paire de clés fonctionnelle pour cette identité, que vous utilisez ensuite directement.

## Générer des clés pour une cible

```bash
aws iam create-access-key --user-name <privileged-user>
# returns AccessKeyId + SecretAccessKey; configure a profile and use them
```

## Notes d'exploitation

- Un utilisateur ne peut détenir qu'au plus deux clés d'accès ; si la cible en a déjà deux, l'opération échoue tant que l'une n'est pas supprimée.
- La nouvelle clé est un identifiant durable qui survit à votre session courante, ce qui en fait aussi un moyen de persistance.
- Aucune modification de politique n'est apportée à la cible, de sorte que l'escalade est discrète du point de vue de l'audit des politiques, mais elle génère tout de même un événement de création de clé.

## Outils

- **AWS CLI** (`iam create-access-key`).
- **Pacu** (`iam__backdoor_users_keys`) : génère des clés sur plusieurs utilisateurs à grande échelle.

## Références

- [Rhino Security Labs: AWS privilege escalation (CreateAccessKey)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
