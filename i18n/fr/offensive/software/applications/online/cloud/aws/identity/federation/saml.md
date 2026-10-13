---
title: "SAML : forger des assertions pour AssumeRoleWithSAML"
order: 1
description: "Forger ou rejouer des assertions SAML approuvées par un fournisseur SAML IAM pour endosser des rôles fédérés via AssumeRoleWithSAML."
keywords:
  - SAML
  - assertion
  - fédération
  - fournisseur IAM
  - AssumeRoleWithSAML
---

# SAML

Un fournisseur d'identité SAML IAM fait confiance aux assertions signées par le certificat d'un IdP externe. `sts:AssumeRoleWithSAML` échange une assertion valide contre des identifiants de rôle. Si vous pouvez signer des assertions (en compromettant la clé de signature de l'IdP) ou rejouer une assertion capturée, vous générez des identifiants pour tout rôle dont vous satisfaites la relation de confiance et les attributs d'assertion.

## Endosser avec une assertion

```bash
aws sts assume-role-with-saml \
  --role-arn arn:aws:iam::<acct>:role/<role> \
  --principal-arn arn:aws:iam::<acct>:saml-provider/<provider> \
  --saml-assertion "$(base64 -w0 assertion.xml)"
```

## Notes d'exploitation

- L'attribut `Role` de l'assertion liste des paires d'ARN rôle/fournisseur ; la relation de confiance du rôle doit nommer le fournisseur SAML comme `Principal.Federated`.
- Le contrôle de la clé de signature de l'IdP est en or : il permet de forger des assertions pour tout rôle associé, l'équivalent SAML d'un golden ticket.
- Les assertions capturées sont rejouables jusqu'à leur expiration : intercepter une assertion en transit suffit donc pour une courte fenêtre.

## Outils

- **AWS CLI** (`sts assume-role-with-saml`).
- **python3-saml** / outillage sur mesure pour créer ou re-signer des assertions.

## Références

- [AWS: SAML 2.0 federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml.html)
- [HackTricks Cloud: AWS federation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
