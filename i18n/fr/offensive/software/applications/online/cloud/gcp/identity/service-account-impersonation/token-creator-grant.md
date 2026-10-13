---
title: "Octroi de Token Creator : vous lier pour usurper un compte de service"
order: 1
description: "Vous accorder roles/iam.serviceAccountTokenCreator sur une cible en appelant iam.serviceAccounts.setIamPolicy, puis en usurpant son identité."
keywords:
  - Token Creator
  - serviceAccounts.setIamPolicy
  - liaison IAM
  - usurpation d'identité
  - compte de service
---

# Octroi de Token Creator

Lorsque vous détenez `iam.serviceAccounts.setIamPolicy` sur un compte de service mais pas encore le droit de l'usurper, vous vous accordez `roles/iam.serviceAccountTokenCreator` sur ce compte, puis générez ses jetons. C'est l'escalade par setIamPolicy réduite à un seul compte de service, et elle ouvre la porte à toutes les techniques d'usurpation d'identité.

## Accorder puis usurper

```bash
gcloud iam service-accounts add-iam-policy-binding <target>@<proj>.iam.gserviceaccount.com \
  --member "user:you@example.com" \
  --role roles/iam.serviceAccountTokenCreator

# now mint the target's token
gcloud auth print-access-token --impersonate-service-account <target>@<proj>.iam.gserviceaccount.com
```

## Notes d'exploitation

- `serviceAccounts.setIamPolicy` sur un compte de service privilégié est en soi un chemin d'escalade complet via cet octroi.
- La liaison est durable : elle persiste jusqu'à sa suppression, un point d'ancrage de persistance en plus d'une escalade.
- Préférez accorder le rôle restreint Token Creator plutôt qu'Owner sur le compte de service afin de rester plus proche d'un IAM d'apparence normale.

## Outils

- **gcloud** (`iam service-accounts add-iam-policy-binding`).

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : usurpation d'identité de compte de service](https://cloud.google.com/iam/docs/service-account-impersonation)
