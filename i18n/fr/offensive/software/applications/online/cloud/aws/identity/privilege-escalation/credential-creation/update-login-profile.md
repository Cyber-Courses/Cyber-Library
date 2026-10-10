---
title: "UpdateLoginProfile : réinitialiser le mot de passe console d'un utilisateur privilégié"
description: "iam:UpdateLoginProfile pour réinitialiser le mot de passe console d'un utilisateur plus privilégié et prendre le contrôle de sa connexion."
keywords:
  - UpdateLoginProfile
  - mot de passe console
  - réinitialisation
  - utilisateur
  - profil de connexion
---

# UpdateLoginProfile

`iam:UpdateLoginProfile` réinitialise le mot de passe console d'un utilisateur disposant déjà d'un profil de connexion. Contre un utilisateur plus privilégié, définissez un mot de passe que vous connaissez et connectez-vous en tant que lui.

## Réinitialiser le mot de passe

```bash
aws iam update-login-profile --user-name <privileged-user> \
  --password 'Newpass123!' --no-password-reset-required
```

## Notes d'exploitation

- C'est perturbant si l'utilisateur se connecte activement, puisque son ancien mot de passe cesse de fonctionner : c'est donc un choix délibéré contre des comptes qui ne sont pas en usage interactif.
- Si la cible n'a pas encore de profil, utilisez plutôt [CreateLoginProfile](create-login-profile.md).
- La MFA console, là où elle est imposée, continue de verrouiller la connexion même après une réinitialisation de mot de passe.

## Outils

- **AWS CLI** (`iam update-login-profile`).
- **Pacu** (`iam__privesc_scan`).

## Références

- [Rhino Security Labs: AWS privilege escalation (UpdateLoginProfile)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
