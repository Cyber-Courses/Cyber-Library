---
title: "CreateLoginProfile : définir un mot de passe console sur un utilisateur qui n'en a pas"
description: "iam:CreateLoginProfile pour définir un mot de passe console sur un utilisateur qui n'en a pas, permettant une connexion interactive en tant que ce principal."
keywords:
  - CreateLoginProfile
  - mot de passe console
  - profil de connexion
  - utilisateur
  - connexion
---

# CreateLoginProfile

Un utilisateur disposant de clés programmatiques mais sans profil de connexion console ne peut pas se connecter à la console web. `iam:CreateLoginProfile` en crée un, de sorte que contre un utilisateur privilégié sans profil, vous créez un mot de passe et vous vous connectez de manière interactive en tant que lui.

## Créer un profil

```bash
aws iam create-login-profile --user-name <privileged-user> \
  --password 'Newpass123!' --no-password-reset-required
# then sign in at the account console URL as that user
```

## Notes d'exploitation

- Échoue si l'utilisateur possède déjà un profil de connexion ; utilisez plutôt [UpdateLoginProfile](update-login-profile.md) pour en réinitialiser un existant.
- `--no-password-reset-required` évite l'invite de changement forcé à la première connexion.
- L'accès console peut atteindre des actions et des vues que le script autour de la CLI rend malaisées, et il porte l'ensemble des autorisations de l'utilisateur.

## Outils

- **AWS CLI** (`iam create-login-profile`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs: AWS privilege escalation (CreateLoginProfile)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
