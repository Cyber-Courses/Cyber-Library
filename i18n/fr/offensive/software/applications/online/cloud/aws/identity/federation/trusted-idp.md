---
title: "IdP de confiance : exploiter la confiance accordée à une identité tierce pour entrer dans AWS"
order: 4
description: "Abuser de la confiance accordée à un fournisseur d'identité tiers ou à un éditeur SaaS pour obtenir un accès AWS fédéré."
keywords:
  - fournisseur d'identité
  - fédération
  - confiance
  - tiers
  - SSO
---

# IdP de confiance

Un compte se fédère souvent à un fournisseur d'identité tiers (Okta, Auth0, Google, un IdP d'entreprise) en tant que fournisseur SAML ou OIDC. La confiance AWS ne vaut que celle de cet IdP externe : compromettez un compte chez l'IdP qui est associé à un rôle AWS, ou l'IdP lui-même, et vous héritez de l'accès AWS associé.

## Cartographier la relation de confiance

```bash
aws iam list-saml-providers
aws iam list-open-id-connect-providers
# for each role, read the trust to see which provider and claims it accepts
aws iam get-role --role-name <role> --query 'Role.AssumeRolePolicyDocument'
```

## Notes d'exploitation

- Le maillon faible est fréquemment l'authentification propre à l'IdP : un compte IdP associé à un rôle et doté d'une MFA faible constitue un chemin vers AWS.
- Des associations de rôle pilotées par un groupe ou un attribut de l'IdP signifient qu'en vous ajoutant à ce groupe côté IdP, vous obtenez le rôle AWS.
- La mécanique concrète suit [SAML](saml.md) ou [OIDC](oidc.md) selon le type de fournisseur.

## Outils

- **AWS CLI** (`iam list-saml-providers` / `list-open-id-connect-providers`).
- Outillage spécifique à l'IdP utilisé.

## Références

- [AWS: identity providers and federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers.html)
- [HackTricks Cloud: AWS federation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
