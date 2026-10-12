---
title: "Fédération"
order: 4
description: "Entrer dans AWS via une identité fédérée : confiance de fournisseurs SAML et OIDC, OIDC GitHub Actions, IdP tiers de confiance et certificats IAM Roles Anywhere."
keywords:
  - fédération
  - SAML
  - OIDC
  - GitHub Actions
  - Roles Anywhere
---

# Fédération

La fédération permet à un fournisseur d'identité externe de forger des identifiants de rôle AWS sans utilisateur IAM. La confiance est déclarée par un objet **identity provider** IAM plus un rôle dont la stratégie de confiance accepte les jetons provenant de lui, restreinte par `Condition` sur des revendications telles que `sub` et `aud`. Lorsque ces conditions sont trop permissives, un attaquant qui contrôle ou peut forger un jeton correspondant assume le rôle.

## Ce qui est regroupé ici

- **[SAML](saml.md)** : `sts:AssumeRoleWithSAML` et confiance de fournisseur SAML trop large.
- **[OIDC](oidc.md)** : `sts:AssumeRoleWithWebIdentity` et conditions `sub`/`aud` faibles sur un fournisseur OIDC.
- **[GitHub Actions](github-actions.md)** : le fournisseur OIDC de GitHub avec une condition `sub` assez permissive pour accepter d'autres dépôts ou branches.
- **[Trusted IdP](trusted-idp.md)** : IdP tiers (Auth0, Okta, Google) de confiance en tant que fournisseurs de web-identity.
- **[Roles Anywhere](roles-anywhere.md)** : confiance de certificat X.509 qui forge des identifiants de rôle en dehors d'AWS.

## Références

- [HackTricks Cloud: AWS federation abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [AWS: web identity federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_oidc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
