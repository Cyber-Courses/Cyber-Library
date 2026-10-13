---
title: "Identifiants GCP"
order: 2
description: "Récupérer des identifiants GCP : le serveur de métadonnées d'instance, Secret Manager, les clés de compte de service, les jetons gcloud et ADC en cache, Cloud KMS, les clés HMAC et les clés API."
keywords:
  - identifiants GCP
  - métadonnées
  - Secret Manager
  - clés de compte de service
  - jetons d'accès
  - KMS
---

# Identifiants

Chaque appel API GCP transporte un identifiant, et le moyen le plus rapide d'élargir son accès est d'en récupérer davantage. Les identifiants se présentent sous forme de **jetons d'accès OAuth** de courte durée (issus du serveur de métadonnées ou de `gcloud`), de **clés JSON de compte de service** de longue durée, et des secrets et clés que les services renvoient lorsqu'on les lit. Chacun a ses propres caractéristiques de vol et de réutilisation, et plusieurs services génèrent de nouveaux identifiants pour quiconque peut les appeler.

## Ce que cette section regroupe

- **[Métadonnées d'instance](instance-metadata/index.md)** : le jeton du compte de service attaché et les clés SSH depuis `metadata.google.internal`, y compris via SSRF.
- **[Secret Manager](secret-manager.md)** : lire les secrets stockés avec `secretmanager.versions.access`.
- **[Clés de compte de service](service-account-keys.md)** : clés JSON exportées, et création de nouvelles clés pour un accès durable.
- **[Jetons d'accès](access-tokens.md)** : piller les identifiants `gcloud` et Application Default Credentials mis en cache sur le disque.
- **[Cloud KMS](cloud-kms.md)** : déchiffrer des données et signer en tant que clé.
- **[Clés HMAC](hmac-keys.md)** : générer une clé HMAC Cloud Storage pour un compte de service.
- **[Clés API](api-keys.md)** : créer ou lister des clés API Google pour un accès durable et non restreint.

## Références

- [HackTricks Cloud : GCP credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google : service account credentials](https://cloud.google.com/iam/docs/service-account-creds)
