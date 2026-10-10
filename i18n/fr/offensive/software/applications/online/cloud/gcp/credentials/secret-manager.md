---
title: "Secret Manager : lire les secrets stockés avec versions.access"
order: 2
description: "Lire les secrets stockés avec secretmanager.versions.access lorsqu'un principal ou un compte de service détient un accès à Secret Manager."
keywords:
  - Secret Manager
  - secretmanager.versions.access
  - secret
  - credential store
  - GCP
  - access
---

# Secret Manager

Secret Manager contient les identifiants que les applications récupèrent à l'exécution : `secretmanager.versions.access` est donc une lecture directe des mots de passe, clés d'API et chaînes de connexion du projet. Cette permission est souvent accordée largement aux comptes de service applicatifs, ce qui en fait une cible de choix dès que vous détenez le jeton d'un tel compte.

## Lister et lire

```bash
gcloud secrets list --format='value(name)'
gcloud secrets versions access latest --secret=<name>
# a specific version
gcloud secrets versions access 3 --secret=<name>
```

## Balayer tout ce qui est lisible

```bash
for s in $(gcloud secrets list --format='value(name)'); do
  echo "== $s =="
  gcloud secrets versions access latest --secret="$s" 2>/dev/null
done
```

## Notes d'exploitation

- Lire un secret ne le fait pas tourner (rotation) : les identifiants de base de données et de tiers récupérés continuent de fonctionner.
- IAM peut être défini par secret : un principal refusé au niveau du projet peut tout de même lire certains secrets individuels ; énumérez avec `gcloud secrets get-iam-policy` là où cela compte.
- Les versions désactivées ou détruites ne renvoient rien, mais des versions activées plus anciennes le font souvent, ce qui permet de récupérer une valeur ayant subi une rotation après exposition.

## Outils

- **gcloud** (`secrets versions access`).
- Scripts de type **GCPGoat** / **gcp_enum** pour balayer en masse les secrets lisibles.

## Références

- [Google: Secret Manager access](https://cloud.google.com/secret-manager/docs/access-secret-version)
- [HackTricks Cloud: GCP Secret Manager](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
