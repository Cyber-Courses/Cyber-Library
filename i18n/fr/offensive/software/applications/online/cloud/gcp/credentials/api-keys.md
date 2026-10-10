---
title: "Clés API : créer et lister les clés API Google"
order: 7
description: "Créer ou lister des clés API Google avec serviceusage.apiKeys.create et list pour un accès API durable et non restreint."
keywords:
  - clés API
  - serviceusage.apiKeys
  - apiKeys.create
  - non restreint
  - persistance
  - API Google
---

# Clés API

Les **clés API** Google sont des chaînes autonomes qui authentifient les appels aux API activées sans identité ni expiration de jeton. `serviceusage.apiKeys.create` en génère une et `serviceusage.apiKeys.list` (avec `getKeyString`) récupère celles qui existent déjà, offrant un accès durable à toutes les API auxquelles la clé est autorisée, souvent sans aucune restriction définie.

## Lister et récupérer des clés existantes

```bash
gcloud services api-keys list --format='value(name,displayName)'
gcloud services api-keys get-key-string <key-id>
```

## Créer une nouvelle clé

```bash
gcloud services api-keys create --display-name="legit-looking"
# returns the key string; unrestricted unless --api-target / --allowed-* is set
```

## Notes d'exploitation

- Les clés non restreintes peuvent appeler n'importe quelle API activée du projet et, comme elles ne sont liées à aucune identité, elles contournent la révocation des jetons IAM, ce qui fait d'une clé créée un moyen de persistance discret.
- Les clés existantes se retrouvent souvent dans des applications mobiles, du JavaScript front-end et des fichiers de configuration : récupérer la chaîne de la clé transforme une fuite de code en accès direct au projet.
- Les clés n'apparaissent pas dans la politique IAM, si bien que les défenseurs qui auditent les bindings passent à côté.

## Outils

- **gcloud** (`services api-keys create / list / get-key-string`).

## Références

- [Google : API keys](https://cloud.google.com/docs/authentication/api-keys)
- [HackTricks Cloud : GCP API keys](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
