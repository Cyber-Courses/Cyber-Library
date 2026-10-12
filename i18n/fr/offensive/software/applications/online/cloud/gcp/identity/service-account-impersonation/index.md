---
title: "Usurpation d'identité de compte de service"
order: 3
description: "Générer des identifiants pour un compte de service cible : getAccessToken, signJwt et signBlob, octrois de Token Creator, délégation implicite et création de clés à longue durée de vie."
keywords:
  - usurpation d'identité de compte de service
  - getAccessToken
  - signJwt
  - Token Creator
  - délégation
  - création de clé
---

# Usurpation d'identité de compte de service

L'usurpation d'identité est l'escalade GCP la plus directe : une permission sur un **compte de service** cible vous permet d'agir en tant que celui-ci sans rien déployer. Le rôle `roles/iam.serviceAccountTokenCreator` (et les permissions individuelles ci-dessous) est le pivot, et il est fréquemment accordé de manière trop large.

## Les primitives

- **[getAccessToken](get-access-token.md)** : générer directement un jeton OAuth à courte durée de vie pour la cible.
- **[signJwt et signBlob](sign-jwt-and-sign-blob.md)** : forger un JWT ou un blob signé en tant que cible, puis l'échanger contre un jeton.
- **[Octroi de Token Creator](token-creator-grant.md)** : vous accorder Token Creator sur la cible avec `serviceAccounts.setIamPolicy`, puis usurper son identité.
- **[Délégation implicite](implicit-delegation.md)** : enchaîner via un compte intermédiaire pour atteindre un compte que vous ne pouvez pas appeler directement.
- **[Création de clé](key-creation.md)** : créer une clé JSON à longue durée de vie pour un accès durable et de la persistance.

## Utiliser une identité usurpée

La plupart des commandes `gcloud` acceptent l'option d'usurpation d'identité une fois que vous détenez la permission :

```bash
gcloud <command> --impersonate-service-account=<target-sa>@<project>.iam.gserviceaccount.com
gcloud auth print-access-token --impersonate-service-account=<target-sa>@...
```

## Références

- [Rhino Security Labs : escalade de privilèges GCP](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud : usurpation d'identité de compte de service GCP](https://hackingthe.cloud/)
- [Google : usurpation d'identité de compte de service](https://cloud.google.com/iam/docs/service-account-impersonation)
