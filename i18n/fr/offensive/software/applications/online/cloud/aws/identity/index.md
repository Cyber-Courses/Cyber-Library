---
title: "Identité AWS"
order: 1
description: "Attaques sur l'identité et les accès AWS : énumération des principaux et politiques IAM, catalogue complet des chemins d'escalade de privilèges, endossement de rôle inter-comptes et fédération, et les couches d'identité Cognito, Identity Center et Organizations."
keywords:
  - IAM
  - AssumeRole
  - PassRole
  - escalade de privilèges
  - fédération
---

# Identité

IAM est l'endroit où se gagnent les attaques AWS. Chaque appel d'API est autorisé en fonction des politiques du principal appelant, si bien que les questions sont toujours les mêmes : que peut faire ce principal, quels rôles peut-il **endosser**, et quelles autorisations lui permettent d'**escalader**. L'escalade de privilèges se situe ici parce que, dans AWS, c'est un problème d'autorisations et non d'exploitation d'hôte : une seule action dangereuse, ou une **politique de confiance** de rôle trop permissive, promeut une identité limitée au rang d'administrateur.

Au-delà des utilisateurs et rôles IAM classiques, AWS expose plusieurs couches d'identité distinctes, chacune porteuse de ses propres chemins d'entrée : le service d'identité grand public **Cognito**, **IAM Identity Center** (SSO) à l'échelle d'une organisation, et **Organizations** lui-même, où un seul rôle endossé atteint chaque compte membre.

## Ce qui se range ici

- **[Énumération](enumeration.md)** : cartographier les principaux, les politiques et les relations de confiance, et résoudre les autorisations effectives du principal courant.
- **[Escalade de privilèges](privilege-escalation/index.md)** : le catalogue complet des chemins IAM vers l'administrateur, de `iam:PassRole` aux réécritures de politiques et de politiques de confiance.
- **[Endossement de rôle](role-assumption/index.md)** : `sts:AssumeRole` entre comptes et abus de deputy confus sur des politiques de confiance permissives.
- **[Fédération](federation/index.md)** : confiance envers les fournisseurs SAML et OIDC, OIDC GitHub Actions, IdP tiers de confiance et IAM Roles Anywhere.
- **[Cognito](cognito/index.md)** : rôles de pool d'identités non authentifié, inscription libre à un pool d'utilisateurs et abus des revendications de jetons.
- **[Identity Center](identity-center/index.md)** : affectation de jeux d'autorisations et magasin d'identités pour atteindre les comptes de toute l'organisation.
- **[Organizations](organizations/index.md)** : le rôle de compte membre créé par le compte de gestion et les politiques de contrôle des services.

L'énumération est intégrée à chaque surface d'AWS, mais le graphe IAM qui pilote l'ensemble est cartographié ici avec **Pacu** et **PMapper**.

## Références

- [Rhino Security Labs: AWS IAM privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [PMapper (NCC Group)](https://github.com/nccgroup/PMapper)
- [HackTricks Cloud: AWS IAM privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/index.html)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
