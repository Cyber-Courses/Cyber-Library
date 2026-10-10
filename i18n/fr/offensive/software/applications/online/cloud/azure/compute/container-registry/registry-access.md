---
title: "Accès au registre : identifiants admin et pull/push sur les dépôts"
description: "Atteindre un Azure Container Registry avec les identifiants du compte admin ou des droits de pull/push sur les dépôts pour lire et altérer les images."
keywords:
  - ACR
  - compte admin
  - dépôt
  - pull
  - push
  - registre
---

# Accès au registre

Quand le **compte admin** ACR est activé, un unique couple nom d'utilisateur et mot de passe accorde un pull et un push complets sur chaque dépôt, et l'identifiant est lisible depuis le plan de gestion par quiconque détient le droit adéquat. Même sans lui, `AcrPull`/`AcrPush` sur le registre vous permet de lire les images (pour les secrets qui y sont intégrés) et d'en pousser d'altérées.

## Récupérer l'identifiant admin et se connecter

```bash
az acr credential show -n <registry>          # username + two passwords
az acr login -n <registry>                    # or docker login <registry>.azurecr.io
az acr repository list -n <registry>
docker pull <registry>.azurecr.io/<repo>:<tag>
```

## Notes d'exploitation

- Récupérez chaque image et parcourez (grep) les couches à la recherche de secrets, de jetons et de chaînes de connexion ; les artefacts de build en laissent fuir en permanence.
- `az acr credential show` fonctionne pour quiconque détient `listCredentials` sur le registre, de sorte que le compte admin est une cible facile même sans Docker sur l'hôte.
- Pousser un tag empoisonné par-dessus une image qu'AKS ou App Service récupère est traité sous [Tasks](tasks.md) et l'empoisonnement d'images.

## Outils

- **Azure CLI** (`az acr credential show`, `az acr repository`).
- **docker** / **crane** / **skopeo** : récupérer, inspecter et pousser des images.

## Références

- [Microsoft : authentification ACR](https://learn.microsoft.com/azure/container-registry/container-registry-authentication)
- [HackTricks Cloud : ACR](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
