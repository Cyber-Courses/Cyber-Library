---
title: "Identités managées"
order: 3
description: "Abuser des identités managées Azure : forger des tokens pour les identités system-assigned et user-assigned liées au compute, aux functions et à d'autres ressources."
keywords:
  - identité managée
  - system-assigned
  - user-assigned
  - IMDS
  - token
  - Azure AD
---

# Identités managées

Une identité managée est un principal de service Entra qu'Azure lie à une ressource et dont la plateforme fait tourner les credentials. Tout code qui s'exécute sur la ressource peut demander au point de terminaison de métadonnées local un token pour cette identité, donc contrôler la ressource revient à agir en tant que l'identité avec tout son RBAC. Les identités managées sont le moteur derrière la plupart des escalades et des déplacements latéraux Azure.

## Les deux types

- **[System-assigned](system-assigned.md)** : créée avec une ressource unique et liée à elle, supprimée avec elle.
- **[User-assigned](user-assigned.md)** : une identité autonome qui peut être attachée à plusieurs ressources, donc sa compromission est réutilisable.

Obtenir le token est une technique de [credentials](../../credentials/instance-metadata/index.md) (le point de terminaison IMDS) ; attacher une identité privilégiée à une ressource que vous contrôlez est l'escalade par [attribution d'identité managée](../privilege-escalation/managed-identity-assignment.md).

## Références

- [SpecterOps : chemins d'attaque des identités managées (BARK)](https://github.com/BloodHoundAD/BARK)
- [Microsoft : présentation des identités managées](https://learn.microsoft.com/entra/identity/managed-identities-azure-resources/overview)
- [HackTricks Cloud : identités managées Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
