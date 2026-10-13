---
title: "Métadonnées d'instance"
order: 1
description: "Lire le serveur de métadonnées GCP (metadata.google.internal) pour obtenir le jeton et les clés SSH du compte de service attaché, y compris via SSRF vers le point de terminaison de métadonnées."
keywords:
  - métadonnées d'instance
  - serveur de métadonnées
  - jeton de compte de service
  - SSRF
  - GCE
  - computeMetadata
---

# Métadonnées d'instance

Chaque instance Compute Engine (ainsi que les nœuds GKE, Cloud Run et les workers Cloud Build) peut joindre le **serveur de métadonnées** à l'adresse `metadata.google.internal` (`169.254.169.254`). Il fournit le jeton OAuth du compte de service attaché, ses scopes accordés, les métadonnées du projet et les clés SSH, sans autre authentification qu'un en-tête de requête obligatoire. Lors d'une prise de pied initiale, c'est la première source d'identifiants ; atteint via une vulnérabilité web, c'est un vecteur de server-side request forgery.

## Ce que cette section regroupe

- **[Jeton de compte de service](service-account-token.md)** : le jeton OAuth et les scopes depuis le point de terminaison des comptes de service, depuis un shell.
- **[SSRF vers les métadonnées](ssrf-to-metadata.md)** : le même point de terminaison atteint via une falsification de requête côté serveur sur une application hébergée.

## L'en-tête obligatoire

Chaque requête de métadonnées nécessite `Metadata-Flavor: Google` ; les requêtes qui en sont dépourvues sont refusées, ce qui complique une SSRF naïve incapable de définir des en-têtes (et ce qu'un `?alt=json` combiné à une lecture récursive permet d'exploiter dès que c'est possible).

```bash
curl -s -H 'Metadata-Flavor: Google' \
  'http://metadata.google.internal/computeMetadata/v1/?recursive=true&alt=json'
```

## Références

- [Google : VM metadata](https://cloud.google.com/compute/docs/metadata/overview)
- [Hacking the Cloud : GCP metadata SSRF](https://hackingthe.cloud/)
- [HackTricks Cloud : GCP metadata](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
