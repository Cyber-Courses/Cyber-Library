---
title: "Élévation de privilèges"
order: 2
description: "Grimper dans Azure RBAC : écrire des attributions de rôle vers Owner, concevoir des rôles personnalisés permissifs, elevate-access vers la racine, attribuer des identités managées privilégiées et déployer des modèles s'exécutant sous une identité supérieure."
keywords:
  - élévation de privilèges Azure
  - roleAssignments write
  - rôle personnalisé
  - elevateAccess
  - identité managée
  - déploiement ARM
---

# Élévation de privilèges

L'escalade Azure est un problème de plan d'autorisation. Un principal qui peut écrire le graphe RBAC, ou qui contrôle une ressource portant une identité managée, se promeut sans toucher à un hôte. Les chemins ci-dessous sont les équivalents côté plan de ressources Azure du catalogue AWS IAM, ancrés dans les recherches MicroBurst de NetSPI et BARK de SpecterOps.

## Les chemins

- **[Écriture d'attribution de rôle](role-assignment-write.md)** : `Microsoft.Authorization/roleAssignments/write` pour se donner Owner ou Contributor à n'importe quel scope.
- **[Définition de rôle personnalisé](custom-role-definition.md)** : `roleDefinitions/write` pour concevoir un rôle avec des actions joker, puis se l'auto-attribuer.
- **[Elevate access](elevate-access.md)** : activer User Access Administrator au scope racine pour atteindre chaque abonnement du tenant.
- **[Attribution d'identité managée](managed-identity-assignment.md)** : attacher une identité managée privilégiée à une ressource que vous contrôlez, puis forger son token.
- **[Modèle de déploiement](deployment-template.md)** : un déploiement ARM ou une ressource deploymentScripts qui s'exécute sous une identité privilégiée.

## Choisir un chemin

Un `roleAssignments/write` est le levier le plus direct quand vous le détenez. Sinon, le contrôle d'une ressource qui porte déjà une identité managée privilégiée (une VM, une Function, un Automation Account) est en général la voie vers le haut : empruntez l'identité via les [identités managées](../managed-identities/index.md) et la surface de compute. BARK énumère lesquels de ces chemins le principal courant peut atteindre.

## Références

- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [SpecterOps : élévation de privilèges Azure avec BARK](https://github.com/BloodHoundAD/BARK)
- [HackTricks Cloud : élévation de privilèges Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
