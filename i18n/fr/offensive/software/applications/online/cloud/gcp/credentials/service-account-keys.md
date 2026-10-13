---
title: "Clés de compte de service : clés JSON exportées et création de nouvelles clés"
order: 3
description: "Rechercher et utiliser des fichiers de clés JSON de compte de service exportés, et créer de nouvelles clés pour un accès durable."
keywords:
  - clé de compte de service
  - clé JSON
  - fichier de clé
  - gcloud auth
  - persistance
  - identifiant
---

# Clés de compte de service

Une **clé JSON** de compte de service est un identifiant hors ligne à longue durée de vie : quiconque détient le fichier s'authentifie en tant que ce compte depuis n'importe où, sans expiration de jeton. Les clés fuitent dans le code source, la configuration CI, les ordinateurs portables des développeurs et les buckets de stockage, et `iam.serviceAccountKeys.create` vous permet d'en générer une nouvelle pour n'importe quel compte que vous pouvez atteindre, ce qui constitue à la fois une escalade et une porte dérobée durable.

## Utiliser une clé trouvée

```bash
gcloud auth activate-service-account --key-file=key.json
gcloud auth print-access-token          # now acting as the service account
```

## Créer une clé pour un compte cible

```bash
# needs iam.serviceAccountKeys.create on the target account
gcloud iam service-accounts keys create loot.json \
  --iam-account=<target>@<project>.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file=loot.json
```

## Rechercher des clés sur un hôte

```bash
grep -rl '"type": "service_account"' / 2>/dev/null
ls ~/.config/gcloud/   # legacy; ADC and key refs live here too
```

## Notes d'exploitation

- Une clé créée est la persistance durable la plus discrète sur GCP : elle survit aux réinitialisations de mot de passe et à la révocation des jetons, et la création de clé est un événement d'audit unique facilement noyé dans le bruit.
- Les clés ne portent pas de scopes, donc une clé pour un compte doté de rôles étendus offre un accès complet, contrairement à un jeton de métadonnées restreint par scope.
- Préférez l'[usurpation d'identité](../identity/service-account-impersonation/index.md) à la création de clé lorsque vous n'avez besoin que d'un accès à court terme et souhaitez laisser moins de traces.

## Outils

- **gcloud** (`iam service-accounts keys create`, `auth activate-service-account`).
- **TruffleHog** / **gitleaks** pour trouver les clés fuitées dans le code source et l'historique.

## Références

- [Google : gestion des clés de compte de service](https://cloud.google.com/iam/docs/keys-create-delete)
- [Rhino Security Labs : escalade de privilèges GCP](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
