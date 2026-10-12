---
title: "Création d'identifiants"
order: 2
description: "Générer de nouveaux identifiants pour un principal IAM sur lequel vous pouvez écrire : clés d'accès supplémentaires et profils de connexion console nouveaux ou réinitialisés."
keywords:
  - CreateAccessKey
  - CreateLoginProfile
  - UpdateLoginProfile
  - clé d'accès
  - profil de connexion
  - identifiants
---

# Création d'identifiants

Si vous pouvez créer ou réinitialiser des identifiants sur un principal plus privilégié, vous devenez ce principal sans toucher à ses politiques. Deux primitives : les **clés d'accès** programmatiques, et les **profils de connexion** console.

## Chemins

- **[CreateAccessKey](create-access-key.md)** : générer un second jeu de clés à long terme pour un utilisateur privilégié.
- **[CreateLoginProfile](create-login-profile.md)** : définir un mot de passe console sur un utilisateur qui n'en a pas.
- **[UpdateLoginProfile](update-login-profile.md)** : réinitialiser le mot de passe console d'un utilisateur privilégié.

## Références

- [Rhino Security Labs: AWS privilege escalation (credential paths)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
