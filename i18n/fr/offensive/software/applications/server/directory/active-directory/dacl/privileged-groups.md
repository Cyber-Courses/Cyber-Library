---
title: "Groupes privilégiés : élévation via l'appartenance à des groupes intégrés"
order: 7
description: "Ce qu'accorde à un attaquant l'appartenance aux groupes privilégiés intégrés d'Active Directory : DnsAdmins, Backup Operators, Server Operators, Print Operators, Account Operators, Schema Admins et d'autres, et l'élévation que chacun permet vers SYSTEM ou Domain Admin."
keywords:
  - groupes privilégiés
  - DnsAdmins
  - Backup Operators
  - Server Operators
  - SeLoadDriverPrivilege
---

# Groupes privilégiés

Plusieurs groupes intégrés d'Active Directory sont **Domain Admin de fait** : leur appartenance accorde un privilège ou un accès qui se convertit directement en SYSTEM sur un contrôleur de domaine ou en accès à la base de données du domaine. *Entrer* dans un tel groupe est un problème de [DACL](index.md) (une ACL faible, une réinitialisation de mot de passe, une [écriture de groupe](group-membership.md)) ; cette page porte sur ce que chaque groupe vous permet de *faire* une fois que vous en êtes membre, là où les chemins d'[énumération des ACL](acl-enumeration.md) et de BloodHound se terminent généralement.

## Les groupes à forte valeur

- **[DnsAdmins](dnsadmins.md)** : charger une DLL dans le service DNS (SYSTEM sur le DC) via `ServerLevelPluginDll`.
- **[Backup Operators](backup-operators.md)** : `SeBackupPrivilege` lit `NTDS.dit` et la ruche SYSTEM au-delà de leurs ACL, chaque empreinte du domaine.
- **[Server Operators](server-operators.md)** : reconfigurer le chemin binaire d'un service pour qu'il s'exécute en tant que LocalSystem sur le DC.

## Les autres, et ce qu'ils accordent

- **Print Operators** : détient **`SeLoadDriverPrivilege`** et peut ouvrir une session sur les DC ; charger un pilote signé malveillant ou vulnérable pour atteindre l'exécution de code noyau/SYSTEM.
- **Account Operators** : gèrent la plupart des utilisateurs et groupes **non protégés** ; utilisez-le pour prendre le contrôle de tout compte non protégé ou ajouter des membres à des groupes non protégés (pas ceux protégés par [AdminSDHolder](adminsdholder.md)). Traité sous [appartenance à un groupe](group-membership.md).
- **Schema Admins** : modifient le **schéma**, y compris le `defaultSecurityDescriptor` d'une classe, de sorte que chaque objet futur de cette classe porte une ACE de votre choix, une porte dérobée lente mais à l'échelle de la forêt.
- **Group Policy Creator Owners** : créent des GPO, ce qui, combiné à un droit de liaison, constitue une exécution de code (voir [Group Policy](../group-policy/index.md)).
- **Pre-Windows 2000 Compatible Access** : accorde une **lecture** étendue sur l'annuaire à ses membres (historiquement Everyone/Anonymous), élargissant l'énumération non authentifiée ou peu privilégiée.
- **Enterprise/Key Admins**, **DHCP Administrators** et des groupes délégués similaires portent chacun des élévations plus étroites mais réelles, à vérifier selon l'environnement.

## Notes d'exploitation

- Ces groupes sont la **fin habituelle d'un chemin BloodHound** : une arête d'ACL mène à l'appartenance, et l'appartenance mène à SYSTEM/DA via la technique ci-dessus, énumérez donc vos appartenances de groupe effectives, y compris imbriquées et par `primaryGroupID`, pas seulement les directes.
- La plupart de ces groupes sont **protégés** (SDProp/AdminSDHolder), vous vous y ajoutez donc rarement via une ACL faible ; vous les atteignez en compromettant un membre existant, à l'exception de **DnsAdmins**, qui n'est **pas** protégé et est fréquemment délégué.
- Une appartenance qui accorde un **privilège** (SeBackup, SeLoadDriver) ne compte que là où vous pouvez ouvrir une session ou agir sur la cible (le DC), confirmez donc que l'ouverture de session/le droit s'y applique.

## Outils

- **BloodHound** : cartographie les arêtes vers ces groupes et signale l'appartenance.
- **PowerView / NetExec** : énumérer l'appartenance effective et les droits des groupes.

## Références

- [HackTricks: privileged groups and token privileges](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html)
- [qazeer notes: operators to Domain Admins](https://notes.qazeer.io/active-directory/exploitation-operators_to_domain_admins)
- [Microsoft: Appendix B, privileged accounts and groups in Active Directory](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory)
