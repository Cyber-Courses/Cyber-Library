---
order: 9
title: "DCShadow : pousser des modifications via un contrôleur de domaine malveillant"
description: "Enregistrer un contrôleur de domaine malveillant pour injecter des modifications arbitraires dans Active Directory via le protocole de réplication (DCShadow), en écrivant du SID history, des appartenances de groupe ou des ACL directement dans l'annuaire tout en contournant la majeure partie de la journalisation."
keywords:
  - DCShadow
  - contrôleur de domaine malveillant
  - DRSUAPI
  - IDL_DRSReplicaAdd
  - persistance de domaine
---

# DCShadow

[DCSync](ntds-and-dcsync.md) abuse de la réplication pour **lire** l'annuaire ; **DCShadow** en abuse pour **écrire**. Vous enregistrez temporairement un **contrôleur de domaine malveillant** dans la configuration, l'utilisez pour pousser des modifications arbitraires (une valeur de SID history, une appartenance de groupe, une ACL) dans un vrai DC via le protocole de réplication, puis le désenregistrez. Comme la modification arrive sous forme de réplication normale DC-à-DC plutôt que d'écriture LDAP ordinaire, elle contourne la majeure partie de la journalisation des changements d'annuaire et de la corrélation SIEM. C'est une primitive de domination de domaine et de persistance, pas un moyen d'entrée : vous avez déjà besoin des droits de réplication.

## Comment ça fonctionne

DCShadow (Mimikatz `lsadump::dcshadow`) s'exécute en deux parties :

1. **Enregistrer et servir** (en tant que **SYSTEM** sur n'importe quel hôte joint au domaine) : Mimikatz ajoute les SPN et les objets `nTDSDSA`/serveur qui font passer l'hôte pour un DC, puis héberge le serveur RPC **MS-DRSR** minimal qui servira les données falsifiées.
2. **Pousser** (en tant qu'utilisateur disposant de droits de réplication, généralement **Domain/Enterprise Admin**) : une seconde session Mimikatz déclenche `IDL_DRSReplicaAdd`, forçant un DC légitime à tirer les modifications préparées depuis le DC malveillant.

```text
# Session 1 (SYSTEM): stage the change and host the rogue DC
lsadump::dcshadow /object:targetUser /attribute:primaryGroupID /value:512
# Session 2 (Domain Admin): force replication to push it
lsadump::dcshadow /push
```

## Ce que vous poussez

N'importe quoi dans l'annuaire, choisi pour être discret et durable :

- **`sIDHistory`** sur un compte que vous contrôlez, injectant un SID privilégié (domain admins, enterprise admins) pour un accès permanent.
- **`primaryGroupID` = 512** pour faire d'un utilisateur un administrateur de domaine sans apparaître dans la liste `member` du groupe.
- Des modifications de **`ntSecurityDescriptor`** (par exemple sur [AdminSDHolder](../../dacl/adminsdholder.md)) pour planter une porte dérobée ACL durable.
- Des changements d'attributs qui annulent les nettoyages des défenseurs, réappliqués à la demande.

## Notes d'exploitation

- Le DC malveillant n'existe que pour le push et est démantelé ensuite, il n'y a donc pas d'objet DC malveillant durable ; ce qui persiste, c'est la **modification** que vous avez injectée.
- Le chemin de réplication signifie que la modification ne génère pas les événements de modification du service d'annuaire qu'une écriture LDAP produirait, ce qui est tout l'intérêt.
- Elle nécessite des droits de réplication (DA/EA, ou un principal disposant de `DS-Replication-Get-Changes` plus les droits `DS-Install-Replica`/d'écriture sur la configuration), elle est donc post-compromission ; associez-la à une lecture [DCSync](ntds-and-dcsync.md) pour savoir exactement quoi écraser.
- Les variantes `minimal`/`DcShadow` réduisent les droits nécessaires dans certaines configurations, mais le cas de niveau DA est le plus fiable.

## Outils

- **Mimikatz** (`lsadump::dcshadow`, `lsadump::dcshadow /push`) : l'implémentation de référence (deux sessions).
- **Impacket** (`ntlmrelayx`/`secretsdump` pour la lecture complémentaire) : reconnaissance de ce qu'il faut écraser.

## Références

- [DCShadow (Benjamin Delpy & Vincent Le Toux)](https://www.dcshadow.com/)
- [MITRE ATT&CK T1207: Rogue Domain Controller](https://attack.mitre.org/techniques/T1207/)
- [Hacking Articles: domain persistence, DCShadow](https://www.hackingarticles.in/domain-persistence-dc-shadow-attack/)
