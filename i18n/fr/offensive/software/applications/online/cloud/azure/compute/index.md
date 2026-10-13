---
title: "Ressources de calcul Azure"
order: 3
description: "Attaquer les ressources de calcul Azure : run command et extensions de VM, identités managées, accès au cluster et identité de nœud AKS, images et tâches du Container Registry, Container Instances et scale sets."
keywords:
  - calcul Azure
  - machines virtuelles
  - run command
  - AKS
  - identité managée
---

# Ressources de calcul

Les ressources de calcul Azure s'attaquent de deux façons : exécuter du code sur une ressource pour atteindre l'hôte, et voler l'**identité managée** que la ressource porte pour atteindre l'abonnement. Une VM, un conteneur ou un cluster sur lequel vous pouvez écrire exécute votre code via une opération de gestion de premier ordre (run command, une extension, `command invoke`), et le jeton généré pour son identité attachée vaut généralement plus que l'hôte.

## Surfaces

- **[Machines virtuelles](virtual-machines/index.md)** : run command, Custom Script Extension, vol d'identité managée et user data.
- **[AKS](aks/index.md)** : identifiants admin, `command invoke` et identité du pool de nœuds.
- **[Container Registry](container-registry/index.md)** : identifiants admin, ACR Tasks sous une identité managée et empoisonnement d'images.
- **[Container Instances](container-instances.md)** : exécuter un conteneur sous une identité attachée.
- **[Scale sets](scale-sets.md)** : run command et extensions sur chaque instance.

## Références

- [HackTricks Cloud : calcul Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : run command des VM Azure](https://learn.microsoft.com/azure/virtual-machines/run-command-overview)
