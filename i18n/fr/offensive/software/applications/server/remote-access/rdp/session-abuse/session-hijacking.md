---
title: "Détournement de session : prendre le contrôle d'une session RDP sans mot de passe"
description: "Sur un hôte Windows, un attaquant de niveau SYSTEM utilise tscon pour se connecter à la session RDP d'un autre utilisateur, déconnectée ou active, et la rattacher à la sienne, sans connaître le mot de passe de la victime. Cela livre la session interactive complète et les privilèges de la victime, une technique puissante de déplacement latéral et d'élévation de privilèges sur les serveurs de terminaux et les hôtes de rebond."
keywords:
  - session hijacking
  - tscon
  - rdp
  - terminal services
  - privilege escalation
---

# Détournement de session

Les services Bureau à distance de Windows permettent à un compte privilégié de connecter une session à une autre, et exécuté en tant que SYSTEM, cela ne nécessite aucune connaissance du mot de passe de l'utilisateur cible. La commande `tscon` rattache une session spécifiée à la session actuelle ; exécutée en tant que SYSTEM (par exemple via un service), elle déplace la session d'un autre utilisateur, déconnectée ou active, sur la connexion de l'attaquant, lui donnant son bureau interactif complet et ses privilèges. Sur les serveurs de terminaux, les hôtes Citrix et les hôtes de rebond où de nombreux utilisateurs ont des sessions, c'est une voie directe pour usurper l'identité d'un utilisateur plus privilégié, y compris les administrateurs de domaine qui ont laissé des sessions déconnectées.

```bash
# enumerate sessions and find a higher-privileged target
query user            # ID, STATE (Active/Disc), USERNAME
# hijack as SYSTEM: create a service that runs tscon to attach the target session
#   (run as SYSTEM so no password is needed)
sc create hijack binpath= "cmd /k tscon <TARGET_SESSION_ID> /dest:<YOUR_SESSION>"
sc start hijack
# or with PsExec -s to get SYSTEM, then: tscon <id> /dest:console
```

## Notes d'exploitation

- La propriété « sans mot de passe » ne tient que lors de l'exécution en tant que SYSTEM ; depuis un shell admin, élevez-vous d'abord vers SYSTEM (un service, `PsExec -s`, ou une tâche planifiée) puis `tscon` rattache la session cible sans invite d'identifiants.
- Visez d'abord les sessions déconnectées d'utilisateurs privilégiés : la session déconnectée d'un administrateur de domaine sur un hôte de rebond est une voie directe vers ses droits, et la détourner est discret (l'utilisateur ne regarde pas activement).
- Détourner une session active déconnectera ou surprendra l'utilisateur en direct, c'est donc plus bruyant ; les sessions déconnectées sont préférables.
- C'est un déplacement latéral et une élévation de privilèges locaux à l'hôte ; cela nécessite un accès admin/SYSTEM existant sur le serveur de terminaux, donc cela s'enchaîne après un accès RDP initial ou autre.

## Outils

- [tscon / query (built-in)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tscon)
- [PsExec (SYSTEM)](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec)

## Références

- [Microsoft: tscon](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tscon)
- [HackTricks: RDP session hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
