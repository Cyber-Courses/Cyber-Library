---
title: "Container Registry"
order: 3
description: "Abuser d'Azure Container Registry : identifiants admin et accès aux dépôts, ACR Tasks exécutées sous une identité managée, et empoisonnement d'images."
keywords:
  - ACR
  - Container Registry
  - identifiants admin
  - ACR Tasks
  - empoisonnement d'images
---

# Container Registry

Azure Container Registry conserve les images qu'AKS, App Service et Container Instances récupèrent et exécutent, ce qui en fait à la fois un magasin d'identifiants et un point d'ancrage dans la chaîne d'approvisionnement. Le compte admin et les droits sur les dépôts donnent un accès en lecture et en écriture aux images ; les **ACR Tasks** exécutent des étapes de build à l'intérieur du registre sous une identité managée.

## Pages

- **[Accès au registre](registry-access.md)** : identifiants admin et pull/push sur les dépôts.
- **[Tasks](tasks.md)** : ACR Tasks exécutées sous une identité managée, et empoisonnement d'images.

## Références

- [HackTricks Cloud : ACR](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : authentification ACR](https://learn.microsoft.com/azure/container-registry/container-registry-authentication)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
