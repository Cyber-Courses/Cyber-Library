---
title: "DACL : transformer les permissions d'objet en contrôle"
order: 2
description: "Attaques sur les DACL Active Directory : la carte complète des entrées de contrôle d'accès exploitables (GenericAll, GenericWrite, WriteDacl, WriteOwner, WriteSPN, AddKeyCredentialLink, ForceChangePassword, AddMember, droits de réplication et de mot de passe géré) et l'outillage Linux et Windows qui transforme chacune en contrôle d'un principal privilégié."
keywords:
  - DACL
  - abus d'ACL
  - GenericAll
  - WriteDacl
  - bloodyAD
---

# DACL

Chaque objet Active Directory porte une liste de contrôle d'accès discrétionnaire (DACL) : une liste d'entrées de contrôle d'accès (ACE) indiquant quels principaux peuvent le lire ou le modifier. Les administrateurs délèguent ces droits généreusement et ne les auditent presque jamais, si bien qu'AD regorge d'ACE qui permettent à un principal ordinaire d'en modifier un privilégié. Le travail sur les DACL consiste à transformer une **écriture** sur un objet en **contrôle** de celui-ci, et BloodHound existe en grande partie pour trouver ces arêtes et les enchaîner (A peut écrire sur B, B est admin de C) en un chemin vers Domain Admin.

## La carte complète des arêtes

Chaque droit exploitable, l'attribut ou le droit de contrôle d'accès qui le sous-tend, et où la technique est documentée :

| Droit (arête BloodHound) | Soutenu par | Se transforme en |
| --- | --- | --- |
| `GenericAll` | contrôle total | tout ce qui suit |
| `GenericWrite` / `WriteProperty` | écritures de propriétés | SPN, key credential, délégation, appartenance |
| `WriteDacl` | `WRITE_DAC` | [vous accorder n'importe quel droit](ownership-and-acl-rewrite.md) |
| `WriteOwner` / `Owns` | `WRITE_OWNER` | [s'approprier la propriété, puis réécrire la DACL](ownership-and-acl-rewrite.md) |
| `ForceChangePassword` | `User-Force-Change-Password` | [réinitialiser le mot de passe](password-reset.md) |
| `AddMember` / `AddSelf` | écrire `member` | [rejoindre un groupe privilégié](group-membership.md) |
| `WriteSPN` | écrire `servicePrincipalName` | [Kerberoasting ciblé](targeted-kerberoasting.md) |
| `AddKeyCredentialLink` | écrire `msDS-KeyCredentialLink` | [shadow credentials](../authentication/kerberos/shadow-credentials.md) |
| `AddAllowedToAct` / `WriteAccountRestrictions` | écrire `msDS-AllowedToActOnBehalfOfOtherIdentity` | [délégation basée sur les ressources](../authentication/kerberos/delegation/resource-based-constrained.md) |
| `DCSync` | `DS-Replication-Get-Changes` + `-All` sur le domaine | [DCSync](../authentication/credentials/ntds-and-dcsync.md) |
| `ReadGMSAPassword` | lire `msDS-ManagedPassword` | [récupérer le mot de passe gMSA](../managed-accounts/gmsa.md) |
| `ReadLAPSPassword` | lire `ms-Mcs-AdmPwd` / `msLAPS-*` | [récupérer le mot de passe LAPS](../managed-accounts/laps.md) |
| `WriteGPLink` | écrire `gPLink` sur une OU | [lier une GPO à l'OU](../group-policy/index.md) |

Le droit ne compte que par ce qu'il vous permet de faire, le tableau est donc la voie rapide : trouvez l'arête dans BloodHound, sautez à la technique.

## Choisir la conversion la plus discrète

Plusieurs arêtes atteignent le même objectif avec des empreintes très différentes, ce qui est la décision pratique une fois que vous détenez une écriture :

- Les **shadow credentials** (là où PKINIT est disponible) et le **Kerberoasting ciblé** sont **non destructifs** : ils ne changent pas le mot de passe de la victime ni n'excluent personne, préférez-les donc à une réinitialisation de mot de passe quand le compte est en usage.
- Une **réinitialisation de mot de passe** est bruyante et perturbatrice (l'utilisateur légitime perd l'accès), gardez-la donc pour les comptes d'ordinateur ou obsolètes, ou quand rien de plus discret n'est disponible.
- La **RBCD** et l'**ajout de droits de réplication** laissent un changement de configuration durable ; **s'ajouter à un groupe** est trivialement visible dans l'appartenance. Pesez la valeur de persistance contre la détectabilité.

## Pages

- **[Énumération des ACL](acl-enumeration.md)** : trouver les ACE exploitables avec BloodHound et depuis Linux.
- **[BloodHound](bloodhound.md)** : collecter et cartographier les chemins d'attaque sur les ACL et les privilèges à travers le domaine.
- **[Réinitialisation de mot de passe](password-reset.md)** : `ForceChangePassword` / `GenericAll` pour prendre le contrôle d'un compte.
- **[Appartenance à un groupe](group-membership.md)** : `AddMember` / `AddSelf` pour rejoindre un groupe privilégié.
- **[Kerberoasting ciblé](targeted-kerberoasting.md)** : `WriteSPN` pour rendre une cible roastable.
- **[Propriété et réécriture d'ACL](ownership-and-acl-rewrite.md)** : `WriteOwner` / `WriteDacl`, et les limites d'Owner Rights qui les contraignent désormais.
- **[AdminSDHolder](adminsdholder.md)** : persistance via les DACL grâce à SDProp.

Une fois qu'une arête vous amène dans un **groupe privilégié** intégré, l'appartenance elle-même est l'élévation :

- **[Groupes privilégiés](privileged-groups.md)** : ce qu'accorde l'appartenance à DnsAdmins, Backup/Server/Print Operators et d'autres.
- **[DnsAdmins](dnsadmins.md)** : une DLL chargée dans le service DNS en tant que SYSTEM.
- **[Backup Operators](backup-operators.md)** : `SeBackupPrivilege` lit NTDS.dit.
- **[Server Operators](server-operators.md)** : reconfigurer un service pour qu'il s'exécute en tant que SYSTEM.

## Outils

- **BloodHound** : analyse de chemins d'ACL transitifs et identification des arêtes.
- **Impacket** (`dacledit.py`, `owneredit.py`) : lire/écrire des ACE et changer la propriété depuis Linux.
- **bloodyAD** : définir le propriétaire, accorder des droits, réinitialiser des mots de passe, ajouter des key credentials et des membres de groupe depuis Linux.
- **NetExec (`nxc`)** : `-M daclread` pour lire les ACE, `-M maq` pour le quota de comptes machine, et `--ntds` une fois qu'un octroi DCSync atterrit, sur LDAP/SMB.
- **PowerView** (`Add-DomainObjectAcl`, `Set-DomainObjectOwner`) : édition d'ACE et de propriété en local.

## Références

- SpecterOps: An ACE Up the Sleeve, and the BloodHound edge reference
- The Hacker Recipes: DACL abuse
