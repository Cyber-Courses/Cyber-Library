---
title: "Identité Azure"
order: 1
description: "Abuser du plan d'autorisation Azure RBAC : énumérer les attributions de rôle, escalader par des écritures de rôle et de rôle personnalisé, elevate-access et l'attribution d'identité managée, et activer des rôles privilégiés via PIM."
keywords:
  - Azure RBAC
  - attribution de rôle
  - élévation de privilèges
  - identité managée
  - PIM
---

# Identité

L'autorisation Azure repose sur **Azure RBAC** : une attribution de rôle lie un principal à un rôle à un scope donné (groupe d'administration, abonnement, groupe de ressources ou ressource). Les attaques ici consistent à transformer une écriture sur le plan d'autorisation, ou le contrôle d'une ressource portant une identité managée, en un rôle supérieur. Il s'agit uniquement du **plan de ressources** Azure ; les attaques sur le tenant et l'annuaire (rôles Entra, utilisateurs, applications) relèvent d'[Entra ID](../../../identity/entra-id/index.md) dans la section Annuaire.

Les mouvements principaux : écrire une attribution de rôle pour se donner Owner, concevoir un rôle personnalisé avec une action joker, activer `elevateAccess` pour s'emparer de User Access Administrator à la racine, et attacher ou emprunter une **identité managée** privilégiée pour forger son token.

## Ce qui se rattache ici

- **[Énumération](enumeration.md)** : cartographier les attributions de rôle, les rôles personnalisés et les identités managées avec Resource Graph, az cli et ROADtools.
- **[Élévation de privilèges](privilege-escalation/index.md)** : écritures de rôle et de rôle personnalisé, elevate-access, attribution d'identité managée et déploiements privilégiés.
- **[Identités managées](managed-identities/index.md)** : les identités system-assigned et user-assigned comme source de token derrière la plupart des escalades Azure.
- **[PIM](pim.md)** : activer des rôles privilégiés éligibles via Privileged Identity Management.

La **récupération du token** d'identité managée depuis le point de terminaison de métadonnées est une technique de credentials et se trouve sous [credentials](../credentials/instance-metadata/index.md) ; cette surface porte sur l'obtention et l'escalade du rôle lui-même.

## Références

- [HackTricks Cloud : élévation de privilèges Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/az-privilege-escalation/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [SpecterOps : chemins d'attaque des identités managées (BARK)](https://github.com/BloodHoundAD/BARK)
