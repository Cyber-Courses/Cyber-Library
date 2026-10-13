---
title: "Rôles Entra"
order: 3
description: "Élever les privilèges d'annuaire dans Entra : abuser des rôles privilégiés tels que Privileged Role Administrator et Application Administrator, activation PIM, et cadrage par unité administrative."
keywords:
  - rôles Entra
  - élévation de privilèges
  - PIM
  - Global Administrator
  - unités administratives
---

# Rôles

Les rôles d'annuaire Entra accordent un pouvoir administratif à l'échelle du tenant, et plusieurs rôles intermédiaires atteignent Global Administrator via une courte chaîne : un rôle qui peut attribuer des rôles, réinitialiser les identifiants d'un utilisateur privilégié, ou ajouter un secret d'application est de fait Global Admin. Cette surface consiste à reconnaître ces rôles et la machinerie d'activation et de cadrage qui les entoure.

## Ce qui est regroupé ici

- **[Élévation de privilèges](privilege-escalation.md)** : les rôles qui s'enchaînent vers Global Administrator.
- **[PIM](pim.md)** : activer les rôles privilégiés éligibles juste à temps.
- **[Unités administratives](administrative-units.md)** : attributions de rôles cadrées et leurs failles.

## Références

- [SpecterOps : AzureHound role attack paths](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud : Entra roles](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : Entra built-in roles](https://learn.microsoft.com/entra/identity/role-based-access-control/permissions-reference)
